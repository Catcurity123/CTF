
# PICOCTF - JaWT Scratchpad




## Description
>Check the admin scratchpad! https://jupiter.challenges.picoctf.org/problem/63090/ or http://jupiter.challenges.picoctf.org:63090

## 1. Solution

The web provide us with an input box and some scattered information regarding JaWT and a github post from the author.

```html
<!doctype html>
<html>

	<title> JaWT - an online scratchpad </title>
	<link rel="stylesheet" href="/static/css/stylesheet.css">
	<body>
		<header><h1>JaWT</h1> <br> <i><small>powered by <a href="https://jwt.io/">JWT</a></small></i></header>
		
		<div id="main">
		
			<article>
				<h1>Welcome to JaWT!</h1>

				<p>
					JaWT is an online scratchpad, where you can "jot" down whatever you'd like! Consider it a notebook for your thoughts. <b style="color:blue "> JaWT works best in Google Chrome for some reason. </b>
				</p>
				
				
				
					<p>
						You will need to log in to access the JaWT scratchpad. You can use any name, other than <code>admin</code>... because the <code>admin</code> user gets a special scratchpad!
					</p>
					<br>
					<form action="#" method="POST">
						<input type="text" name="user" id="name">
					</form>
					<br>

				

				<h2> Register with your name! </h2>

				<p>
					You can use your name as a log in, because that's quick and easy to remember! If you don't like your name, use a short and cool one like <a href="https://github.com/magnumripper/JohnTheRipper">John</a>!
				</p>

			</article>
			<nav></nav>
			<aside></aside>
				
		</div>
		<script> window.onload = function() { document.getElementById("name").focus(); }; </script>
	</body> 
</html>
```

The provided source code doesn't help that much, static analysis would not help ether, let's enter some information on `burpsuite` to see what happen.

#### 1.1 From the provided information, we know that the username should be `admin` so let's try that first.

![admin](https://github.com/Catcurity123/CTF/blob/main/picture/admin.png)

It seems like we can not simply enter admin as our username

#### 1.2 Let's try other username 

![luan](https://github.com/Catcurity123/CTF/blob/main/picture/luan.png)

Other username such as `luan` or `John` work just fine, when taking a look at the request and response of the web we notice some interesting information.

![luanJWT](https://github.com/Catcurity123/CTF/blob/main/picture/luanJWT.png)

Apart from the `cloudfare bot management cookie`, we can also see a `JWT` or `JSON Web Token`. From previous challenges, we know that `JWT` allows the client to indicate its identity for further exchange of information after authentication. Using [jwt.io](jwt.io) we debug the token to see its information.

![Jwt.io](https://github.com/Catcurity123/CTF/blob/main/picture/Jwt.io.png)

How about forging another token with the username `admin` in [Jwt.io](jwt.io)?

![adminjwt](https://github.com/Catcurity123/CTF/blob/main/picture/adminJWT.png)

![testweb](https://github.com/Catcurity123/CTF/blob/main/picture/webtest.png)

This would not work as we did not supply our JWT with the `secret code` from the server. This happen because the signature of `JWT` is crafted using the combination of our session data, current timestampt and the server's secret key. This is quite a secured cryptography as there is no way we can know the server's sercet code.

However, if the secret code is fairly simple, let's say `123456` or `iloveyou` then we can brute force it and the web actually give us a hint of how to do so using [john-the-ripper](https://github.com/openwall/john). This is fairly simple as we just need to provide the password cracker with a `wordlist` and our `Jwt` taken from the valdi username.

![crack](https://github.com/Catcurity123/CTF/blob/main/picture/crack.png)

So we know that the server's secret code is `ilovepico`, let's forge our `jwt` using `jwt.io`

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4ifQ.gtqDl4jVDvNbEe_JYEZTN19Vx6X9NNZtRVbKPBkhO-s
```

Finally we can use `burpsuite` to send this `jwt` to the server and obtain the flag.

![flag](https://github.com/Catcurity123/CTF/blob/main/picture/flag.png)



## 2.Flag

```
picoCTF{jawt_was_just_what_you_thought_f859ab2f}
```

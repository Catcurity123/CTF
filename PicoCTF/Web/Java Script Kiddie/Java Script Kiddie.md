
# PICOCTF - Java Script Kiddie




## Description
>The image link appears broken... https://jupiter.challenges.picoctf.org/problem/58112 or http://jupiter.challenges.picoctf.org:58112

## 1. Reconaissance and Code review

The link directs us to a page with a text input and a button. When we provide the box with some input, a broken image is provided.

Let's take a look at the source code.

```html
<html>
	<head>    
		<script src="jquery-3.3.1.min.js"></script>
		<script>
			var bytes = [];
			$.get("bytes", function(resp) {
				bytes = Array.from(resp.split(" "), x => Number(x));
			});

			function assemble_png(u_in){
				var LEN = 16;
				var key = "0000000000000000";
				var shifter;
				if(u_in.length == LEN){
					key = u_in;
				}
				var result = [];
				for(var i = 0; i < LEN; i++){
					shifter = key.charCodeAt(i) - 48;
					for(var j = 0; j < (bytes.length / LEN); j ++){
						result[(j * LEN) + i] = bytes[(((j + shifter) * LEN) % bytes.length) + i]
					}
				}
				while(result[result.length-1] == 0){
					result = result.slice(0,result.length-1);
				}
				document.getElementById("Area").src = "data:image/png;base64," + btoa(String.fromCharCode.apply(null, new Uint8Array(result)));
				return false;
			}
		</script>
	</head>
	<body>

		<center>
			<form action="#" onsubmit="assemble_png(document.getElementById('user_in').value)">
				<input type="text" id="user_in">
				<input type="submit" value="Submit">
			</form>
			<img id="Area" src=""/>
		</center>

	</body>
</html>
```


#### 1.1 The provided javascript code is extremely obfuscated, let's rewrite it a little bit.
```javascript
var bytes = [];
      //take the bytes file 
			$.get("bytes", function(resp) {
				bytes = Array.from(resp.split(" "), x => Number(x));
			});
      
      //scramble the png file 
			function assemble_png(user_input){
				var KEY_LENGTH = 16;
				var key = "0000000000000000"; //initiate key as a string
				var shifter; //declare shifter

        //check user input equal to 16
				if(user_input.length == KEY_LENGTH){
					key = user_input;
				}

        //scramble value
				var result = [];
        //get the integer value of key at i
				for(var key_position = 0; key_position < KEY_LENGTH; key_position++){
          //make shifter calculatable by storing them as integer
					shifter = key.charCodeAt(key_position) - 48;

          //loop (bytes length / 16) times => making (byte length / 16) blocks of 16 value and assign them base on the shifter
					for(var j = 0; j < (bytes.length / KEY_LENGTH); j ++){
						var result_index = (j *  INPUT_LENGTH) + key_position;
            var bytes_index = (((j + shifter) *  INPUT_LENGTH) % bytes.length) + key_position;
						result[result_index] = bytes[bytes_index]
					}
				}
        // get rid of the remaining 0
				while(result[result.length-1] == 0){
					result = result.slice(0,result.length-1);
				}
        // encode the picture using base64
				document.getElementById("Area").src = "data:image/png;base64," + btoa(String.fromCharCode.apply(null, new Uint8Array(result)));
				return false;
			}
```
#### 1.2 Before diving in the code. let's take an overview of what the code actually does:

![Picture scramble](https://github.com/Catcurity123/CTF/blob/main/picture/picture%20scramble.png?raw=true)

The picture is taken from [this video](https://www.youtube.com/watch?v=ltLA-q3FLvI), as we can see, first we have our original picture. The javascript code will scramble our original data and that is the reason why we got a broken image.

Let's take a look at the PNG format.

![PNG format](https://github.com/Catcurity123/CTF/blob/main/picture/PNG%20format.png)

As our picture is also PNG, therefore, the first 16 bytes of our picture must be:

```
89 50 4E 47 0D 0A 1A 0A 00 00 00 0D 49 48 44 52
```

With that being said let's return to our javascript code.

#### 1.3 code review

``` javascript
var bytes = [];
      //take the bytes file 
			$.get("bytes", function(resp) {
				bytes = Array.from(resp.split(" "), x => Number(x));
			});
```

We can see that the variable `bytes` takes data from somewhere in the web, we can find the data source via `developer tools`:

![Data Source](https://github.com/Catcurity123/CTF/blob/main/picture/data%20source.png)

We can also visit https://jupiter.challenges.picoctf.org/problem/58112/bytes to view the given bytes.

![Bytes](https://github.com/Catcurity123/CTF/blob/main/picture/bytes.png)

When translating the given `decimal` bytes to `hexadecimal` we have the following hex data:

![Hex](https://github.com/Catcurity123/CTF/blob/main/picture/Hex.png)

Using `HxD`, hex editor, we can clearly see the PNG header in 1.2:

![Hex iden](https://github.com/Catcurity123/CTF/blob/main/picture/Hex_iden.png)

The problem can now be over if we just enter `4894748485167104`; however, for the benefit of understanding this problem, let's continue.

``` javascript
function assemble_png(user_input){
				var KEY_LENGTH = 16;
				var key = "0000000000000000"; //initiate key as a string
				var shifter; //declare shifter

				if(user_input.length == KEY_LENGTH){
					key = user_input;
				}
```
The code above is fairly simple, it declares some variable that we will use later on and check if user input is equal to 16.

``` javascript
    var result = [];
    //loop 16 times horizontally
	for(var key_position = 0; key_position < KEY_LENGTH; key_position++){
    //make shifter calculatable by storing them as integer
	shifter = key.charCodeAt(key_position) - 48;

    //loop (bytes length / 16) times => making (byte length / 16) blocks of 16 value and assign them base on the shifter
    //According to the bytes, we will loop 42 times vertically
	for(var j = 0; j < (bytes.length / KEY_LENGTH); j ++){
	var result_index = (j *  INPUT_LENGTH) + key_position;
    var bytes_index = (((j + shifter) *  INPUT_LENGTH) % bytes.length) + key_position;
	result[result_index] = bytes[bytes_index]
 }
}
```
This block of code is the main part of the scambling function, we will loop 16 times horizontally and 42 times vertically.

When the vertical loop occurs, each of the 16 inputs that the user provides will be store as an `integer` which will be our `shifter`. This happens because the user input is rendered as `string`, but we will need it to be `integer` so as to perform calculation on it.
The translation of `string` to `integer` is fairly simple. As the input is `string`, we can take the ASCII decimal value of it and minus by 48:

```
user_input = "1" //which is 49 in decimal
shifter = 1  //which is 1 in decimal
```

When the horizontal loop occurs, each of the 42 bytes on the column will be chosen to form the `result`. Let's take a look at the following visuallization from [this write-up](https://medium.com/@radekk/picoctf-2019-writeup-for-js-kiddie-7af4f0a20838)

![Visualization](https://github.com/Catcurity123/CTF/blob/main/picture/visualization.gif)

As we can see, the `result index` is assigned vertically, therefore, the result will be assigned as follows:

```
First loop: 0,16,32,48,....
Second loop: 1,17,33,49,....
Third loop: 2,18,34,50,....
....
Sixtenth loop: 15,31,47,63,....
```

The `result` will be appended vertically based on the `shifter` of `byte_index` till the end of the provided bytes.

``` javascript
while(result[result.length-1] == 0){
					result = result.slice(0,result.length-1);
				}
        // encode the picture using base64
				document.getElementById("Area").src = "data:image/png;base64," + btoa(String.fromCharCode.apply(null, new Uint8Array(result)));
				return false;
			}
```

The last part of the code is not so difficult, it will remove any redundant 0s and encrypt the `original picture` using `base64` and the `result` that we got from the code above.
## 2. Solution

``` Python
import base64
#know_bytes are the header of PNG file
known_bytes = [0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A, 0x00, 0x00, 0x00, 0x0D, 0x49, 0x48, 0x44, 0x52] 

#Bytes taken from the web
ciphered_bytes = [128,252,182,115,177,211,142,252,189,248,130,93,154,0,68,90,131,255,204,170,239,167,18,51,233,43,0,26,210,72,95,120,227,7,195,126,207,254,115,53,141,217,0,11,118,192,110,0,0,170,248,73,103,78,10,174,208,233,156,187,185,65,228,0,137,128,228,71,159,10,111,10,29,96,71,238,141,86,91,82,0,214,37,114,7,0,238,114,133,0,140,0,38,36,144,108,164,141,63,2,69,73,15,65,68,0,249,13,0,64,111,220,48,0,55,255,13,12,68,41,66,120,188,0,73,27,173,72,189,80,0,148,0,64,26,123,0,32,44,237,0,252,36,19,52,0,78,227,98,88,1,185,1,128,182,177,155,44,132,162,68,0,1,239,175,248,68,91,84,18,223,223,111,83,26,188,241,12,0,197,57,89,116,96,223,96,161,45,133,127,125,63,80,129,69,59,241,157,0,105,57,23,30,241,62,229,128,91,39,152,125,146,216,91,5,217,16,48,159,4,198,23,108,178,199,14,6,175,51,154,227,45,56,140,221,0,230,228,99,239,132,198,133,72,243,93,3,86,94,246,156,153,123,1,204,200,233,143,127,64,164,203,36,24,2,169,121,122,159,40,4,25,64,0,241,9,94,220,254,221,122,8,22,227,140,221,248,250,141,66,78,126,190,73,248,105,5,14,26,19,119,223,103,165,69,177,68,61,195,239,115,199,126,61,41,242,175,85,211,11,5,250,93,79,194,78,245,223,255,189,0,128,9,150,178,0,112,247,210,21,36,0,2,252,144,59,101,164,185,94,232,59,150,255,187,1,198,171,182,228,147,73,149,47,92,133,147,254,173,242,39,254,223,214,196,135,248,34,146,206,63,127,127,22,191,92,88,69,23,142,167,237,248,23,215,148,166,59,243,248,173,210,169,254,209,157,174,192,32,228,41,192,245,47,207,120,139,28,224,249,29,55,221,109,226,21,129,75,41,113,192,147,45,144,55,228,126,250,127,197,184,155,251,19,220,11,241,171,229,213,79,135,93,49,94,144,38,250,121,113,58,114,77,111,157,146,242,175,236,185,60,67,173,103,233,234,60,248,27,242,115,223,207,218,203,115,47,252,241,152,24,165,115,126,48,76,104,126,42,225,226,211,57,252,239,21,195,205,107,255,219,132,148,81,171,53,79,91,27,174,235,124,213,71,221,243,212,38,224,124,54,77,248,252,88,163,44,191,109,63,189,231,251,189,242,141,246,249,15,0,2,230,7,244,161,31,42,182,219,15,221,164,252,207,53,95,99,60,190,232,78,255,197,16,169,252,100,164,19,158,32,189,126,140,145,158,116,245,68,94,149,111,252,74,135,189,83,74,71,218,99,220,208,87,24,228,11,111,245,1,0,98,131,46,22,94,71,244,22,147,21,83,155,252,243,90,24,59,73,247,223,127,242,183,251,124,28,245,222,199,248,122,204,230,79,219,147,11,225,202,239,24,132,55,89,221,143,151,137,63,150,79,211,8,16,4,60,63,99,65,0,2]

#Array to store possible value
possible_key_values = []

#loop horizontally 16 times
for key_pos in range(16):
  #take each of the know_byte out to compare
  known_byte = known_bytes[key_pos]
   
  #Create an array to store the value
  key_possibilities = []
  possible_key_values.append(key_possibilities)
  
  #Create indices of bytes corresponding to the know_bytes
  target_indices = [index for index, b in enumerate(ciphered_bytes) if b == known_byte]
  
  #Append the similar value
  for possible_key_value in range(10):
    #The logic is from the javascript file
    byte_index = (possible_key_value* 16 ) % len(ciphered_bytes) + key_pos
    if byte_index in target_indices:
      key_possibilities.append(possible_key_value)

print(possible_key_values)
#[[4], [8], [9], [4], [7], [4], [8], [4], [8], [5, 6], [1, 2], [6], [7], [1], 
```
With the code above we will have 4 possible answer to the problem:

```
4894748485167104
4894748485267104
4894748486167104
4894748486167104
```

This happens because the given bytes contain duplicate value, and we don't really know which one is the correct value.

![Hex_iden2](https://github.com/Catcurity123/CTF/blob/main/picture/Hex_iden2.png)

Using the art of brute-forcing, `4894748485167104` is the correct answer and we will be greeted with a QR code that translate to:

```
picoCTF{7b15cfb95f05286e37a22dda25935e1e}
```
## 3.Flag
```
picoCTF{7b15cfb95f05286e37a22dda25935e1e}
```
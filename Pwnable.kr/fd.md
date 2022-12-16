
# PWNABLE.KR - fd



### Description:
> Mommy! what is a file descriptor in Linux?

> try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link: https://youtu.be/971eZhMHQQw

> ssh fd@pwnable.kr -p2222 (pw:guest)


### Solution:
Before jumping right into the problem, since this is the first post about pwnable.kr, I would like to set something up:

To download the file on remote server onto local computer, use the following command:

```
scp -r -P2222 fd@pwnable.kr:~/ ./
```

We will then have `fd` and `fd.c` on our computer. Let's take a look at the `C` file:

``` C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

1. The main objective of this problem is to run the `if` statement. Let's analyze each block of the code.

```C
if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
```
The block of code above means that we have to provide the program with an input before starting the program, `argc` stands for `argument count` and `argv` stads for `argument vector`. Therefore, we can run the program via the code below:

```
./fd 1234
```

After entering our initial input, the program will calculate the `file descriptor` or `fd` by changing our input from a string to an interger via the function [atoi](https://www.geeksforgeeks.org/write-your-own-atoi/), then our initial input will be minus by `0x1234`.

```C
int fd = atoi( argv[1] ) - 0x1234;

// argv[1] = "123", then atoi will change it to interger and subtract it by 0x1234
```

Then the program will [read](https://www.geeksforgeeks.org/input-output-system-calls-c-create-open-close-read-write/) `fd` and store it onto `buf`, and it will read `32` character. 

```C
len = read(fd, buf, 32);

// read 32 character from fd and store it onto buf
```

After that, if `buf` is equal to `LETMEWIN` then the program will print out our flag.

``` C
if(!strcmp("LETMEWIN\n", buf)){                
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
```
2. It is worth noticing that if `fd` is `Not 0` it will automatically read the `input` without letting the user to enter any other input. Therefore, `fd` need to be `0` so that we can enter `LETMEWIN`.

```
./fd 4660

# 4660 = 0x1234
```
By doing this `fd` will be `0x1234` and it will be `0` after subtracting. Therefore, we can enter `LETMEWIN` to run the flag.

```
./fd 4660
LETMEWIN
```
3. Our output will be as follows:

```
good job :)
mommy! I think I know what a file descriptor is!!
```
## Flag

```
mommy! I think I know what a file descriptor is!!
```

# PICOCTF - Client-side-again



### Description:
> Can you break into this super secure portal? https://jupiter.challenges.picoctf.org/problem/37821/ (link) or http://jupiter.challenges.picoctf.org:37821


### Solution:
The source code is not very interesting but there is a `javascript` embeded in the `html` file. After using [JavaScript Deobfuscator](https://deobfuscate.io/), we can see that the `verify` function is particularly crucial in this problem.

```javascript
function verify() {
  checkpass = document[_0x4b5b("0x0")]("pass")[_0x4b5b("0x1")];
  split = 4;
  if (checkpass[_0x4b5b("0x2")](0, split * 2) == _0x4b5b("0x3")) {                             // 0 -> 8  picoCTF{
    if (checkpass[_0x4b5b("0x2")](7, 9) == "{n") {                                             // 7 -> 9   {n
      if (checkpass[_0x4b5b("0x2")](split * 2, split * 2 * 2) == _0x4b5b("0x4")) {             // 8 -> 12 not_this
        if (checkpass[_0x4b5b("0x2")](3, 6) == "oCT") {                                        // 3 -> 6
          if (checkpass[_0x4b5b("0x2")](split * 3 * 2, split * 4 * 2) == _0x4b5b("0x5")) {     //24 -> 32 37115}
            if (checkpass.substring(6, 11) == "F{not") {
              if (checkpass[_0x4b5b("0x2")](split * 2 * 2, split * 3 * 2) == _0x4b5b("0x6")) { //16 -> 24 _again_3
                if (checkpass[_0x4b5b("0x2")](12, 16) == _0x4b5b("0x7")) {                     //12 -> 16  this
                  alert(_0x4b5b("0x8"));
                }
              }
            }
          }
        }
      }
    }
  } else {
    alert(_0x4b5b("0x9"));
  }
}
```

We can see that `_0x4b5b` is used to obfuscate different value. We can use `Console developer tools` to evaluate the value.

```
>>>_0x4b5b(0x3)
"picoCTF{"

```
We can do this for all the element in `verify` function. We have the following value

```
(0-8)      == "picoCTF{"
(7-9)      == "{n"
(9-16)     == "not_this"
(3-6)      == "oCT"
(25-32)    == "37115}"
(6-11)     == "F{not"
(17-24)    == "_again_6"
(12-16)    == "this"
```

We have our output as follows:

```
picoCTF{not_this_again_337115}
```
## Flag

```
picoCTF{not_this_again_337115}
```
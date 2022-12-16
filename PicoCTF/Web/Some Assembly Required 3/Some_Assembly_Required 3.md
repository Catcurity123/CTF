
# PICOCTF - Some Assembly Required 3


### Description:
> http://mercury.picoctf.net:60154/index.html
### Solution:
1. After having a look at the `rTEuOmSfG3.js` file, this block of code seem interesting:

``` Javascript
}(_0x143f, 970828));
let exports;
(async () => {
  const _0x484ae0 = _0x187e;
  let _0x487b31 = await fetch("./qCCYI0ajpD"), _0x5eebfd = await WebAssembly[_0x484ae0(293)](await _0x487b31[_0x484ae0(288)]()), _0x30f3ed = _0x5eebfd.instance;
  exports = _0x30f3ed[_0x484ae0(291)];
})();
```

We can then download the web `WebAssembly` file via:

```
wget http://mercury.picoctf.net:60154/qCCYI0ajpD -q -O script.wasm 
```

The `wasm` code seems unreadable, therefore, we can further de-compile it to the following pseudo-code:

```
wasm-decompile script.wasm -o script.dcmp 
```
We can then read the wasm with ease:

```
export memory memory(initial: 2, max: 0);

global g_a:int = 66864;
export global input:int = 1072;
export global key:int = 1067;
export global dso_handle:int = 1024;
export global data_end:int = 1328;
export global global_base:int = 1024;
export global heap_base:int = 66864;
export global memory_base:int = 0;
export global table_base:int = 1;

table T_a:funcref(min: 1, max: 1);

data d_nAb416(offset: 1024) = 
  "\9dn\93\c8\b2\b9A\8b\9f\90\8cb\c5\c3\95\884\c8\93\92\88?\c1\92\c7\db?\c8"
  "\9e\c7\891\c6\c5\c9\8b6\c6\c6\c0\90\00\00";
data d_b(offset: 1067) = "\f1\a7\f0\07\ed";

export function wasm_call_ctors() {
}

export function strcmp(a:int, b:int):int {
  var c:int = g_a;
  var d:int = 32;
  var e:int = c - d;
  e[6]:int = a;
  e[5]:int = b;
  var f:int = e[6]:int;
  e[4]:int = f;
  var g:int = e[5]:int;
  e[3]:int = g;
  loop L_b {
    var h:ubyte_ptr = e[4]:int;
    var i:int = 1;
    var j:int = h + i;
    e[4]:int = j;
    var k:int = h[0];
    e[11]:byte = k;
    var l:ubyte_ptr = e[3]:int;
    var m:int = 1;
    var n:int = l + m;
    e[3]:int = n;
    var o:int = l[0];
    e[10]:byte = o;
    var p:int = e[11]:ubyte;
    var q:int = 255;
    var r:int = p & q;
    if (r) goto B_c;
    var s:int = e[11]:ubyte;
    var t:int = 255;
    var u:int = s & t;
    var v:int = e[10]:ubyte;
    var w:int = 255;
    var x:int = v & w;
    var y:int = u - x;
    e[7]:int = y;
    goto B_a;
    label B_c:
    var z:int = e[11]:ubyte;
    var aa:int = 255;
    var ba:int = z & aa;
    var ca:int = e[10]:ubyte;
    var da:int = 255;
    var ea:int = ca & da;
    var fa:int = ba;
    var ga:int = ea;
    var ha:int = fa == ga;
    var ia:int = 1;
    var ja:int = ha & ia;
    if (ja) continue L_b;
  }
  var ka:int = e[11]:ubyte;
  var la:int = 255;
  var ma:int = ka & la;
  var na:int = e[10]:ubyte;
  var oa:int = 255;
  var pa:int = na & oa;
  var qa:int = ma - pa;
  e[7]:int = qa;
  label B_a:
  var ra:int = e[7]:int;
  return ra;
}

export function check_flag():int {
  var a:int = 0;
  var b:int = 1072;
  var c:int = 1024;
  var d:int = strcmp(c, b);
  var e:int = d;
  var f:int = a;
  var g:int = e != f;
  var h:int = -1;
  var i:int = g ^ h;
  var j:int = 1;
  var k:int = i & j;
  return k;
}

function copy(a:int, b:int) {
  var c:int = g_a;
  var d:int = 16;
  var e:int_ptr = c - d;
  e[3] = a;
  e[2] = b;
  var f:int = e[3];
  if (eqz(f)) goto B_a;
  var g:int = 4;
  var h:int = e[2];
  var i:int = 5;
  var j:int = h % i;
  var k:ubyte_ptr = g - j;
  var l:int = k[1067];
  var m:int = 24;
  var n:int = l << m;
  var o:int = n >> m;
  var p:int = e[3];
  var q:int = p ^ o;
  e[3] = q;
  label B_a:
  var r:int = e[3];
  var s:byte_ptr = e[2];
  s[1072] = r;
}
```
2. The `check_flag` and `strcmp` function are not different from the previous challenge, we only need to take a loot at the `copy` function. Using [Diffchecker](https://www.diffchecker.com/), we can spot the differences between `copy` function from `Some Assembly Reqired 2` with our current challenge:

```
 var g:int = 4;
  var h:int = e[2];
  var i:int = 5;
  var j:int = h % i;
  var k:ubyte_ptr = g - j;
  var l:int = k[1067];
  var m:int = 24;
  var n:int = l << m;
  var o:int = n >> m;
  var p:int = e[3];
  var q:int = p ^ o;
  e[3] = q;
  label B_a:
  var r:int = e[3];
  var s:byte_ptr = e[2];
  s[1072] = r;
```
From the code in `check_flag` and the `global offset`, we know that `e[2]` is our key stored at `offset: 1067` and `e[3]` is the flag stored at `offset 1024`. Therefore, we can translate the new code in `copy` to `Python` as follows:

```python
arr_1067 = [
  0xf1, 0xa7, 0xf0, 0x07, 0xed, 
]

def encode(char, index):
    assert(len(arr_1067) == 5)
    var_j =  4 - (index % len(arr_1067))
    var_l = arr_1067[var_j]
    var_n = ctypes.c_int32(var_l << 24).value
    var_o = ctypes.c_int32(var_n >> 24).value
    var_q = ctypes.c_int32(ord(char) ^ var_o).value
    res = ctypes.c_uint8(var_q).value
    return res
```

3. We can then add our flag at `offset: 1024` to brute force the flag:

```python
import string
import ctypes

arr_1067 = [
  0xf1, 0xa7, 0xf0, 0x07, 0xed, 
]

def encode(char, index):
    assert(len(arr_1067) == 5)
    var_j =  4 - (index % len(arr_1067))
    var_l = arr_1067[var_j]
    var_n = ctypes.c_int32(var_l << 24).value
    var_o = ctypes.c_int32(var_n >> 24).value
    var_q = ctypes.c_int32(ord(char) ^ var_o).value
    res = ctypes.c_uint8(var_q).value
    return res
    
arr_1024 = [
  0x9d, 0x6e, 0x93, 0xc8, 0xb2, 0xb9, 0x41, 0x8b, 0x9f, 0x90, 0x8c, 0x62, 
  0xc5, 0xc3, 0x95, 0x88, 0x34, 0xc8, 0x93, 0x92, 0x88, 0x3f, 0xc1, 0x92, 
  0xc7, 0xdb, 0x3f, 0xc8, 0x9e, 0xc7, 0x89, 0x31, 0xc6, 0xc5, 0xc9, 0x8b, 
  0x36, 0xc6, 0xc6, 0xc0, 0x90, 0x00, 0x00, 
]

for i in range(len(arr_1024)):
    for c in string.printable:
        if encode(c, i) == arr_1024[i]:
            print(c, end = "")

print("")
```

4. We got our output as follows:

```
picoCTF{8aae5dde384ce815668896d66b8f16a1}
```
## Flag

```
picoCTF{8aae5dde384ce815668896d66b8f16a1}
```
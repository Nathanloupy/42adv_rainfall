# Level 00 Walkthrough

## Binary analysis

Decompiling shows the program checks if the argument is 423. If true, it spawns a shell as the next user.

Example decompiled main:

```
08048ec0    int32_t main(int32_t argc, char** argv, char** envp)
08048ede        if (atoi(argv[1]) != 423)
08048f7b            _IO_fwrite("No !\n", 1, 5, _IO_stderr)
08048ede        else
08048eec            char* var_20 = __strdup("/bin/sh")
08048ef0            int32_t var_1c_1 = 0
08048ef8            int32_t eax_5 = __getegid()
08048f01            int32_t eax_6 = __geteuid()
08048f21            __setresgid(eax_5, eax_5, eax_5)
08048f3d            __setresuid(eax_6, eax_6, eax_6)
08048f51            execv("/bin/sh", &var_20)
08048f51        
08048f86        return 0
```

## Exploitation

Run the binary with argument 423 to get a shell as the next user:

```
./level0 423
cat /home/user/level1/.pass
```

Copy the password for the next level.
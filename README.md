# Rainfall Walkthrough

Rainfall is a wargame-style project inspired by the OverTheWire challenges, designed to teach and test binary exploitation skills. Each level presents a vulnerable binary that must be exploited to retrieve the password for the next level.

## Project Structure

- `level0/` to `level9/` : Main levels, each with a `walkthrough.md` and a `flag` file containing the password for the next level.
- `bonus0/` to `bonus3/` : Bonus levels, with their own walkthroughs and flags.
- Each `walkthrough.md` explains the vulnerability, the exploitation steps, and the commands used to solve the level.

## How to Use

1. **Read the walkthroughs**: Each level and bonus has a detailed walkthrough in English, explaining the binary analysis, exploitation, and the commands/scripts used.
2. **Practice**: Try to solve each level yourself before reading the solution for a better learning experience.
3. **Flags**: After exploiting a level, you will obtain the password (flag) for the next one, stored in the corresponding `flag` file for reference.

## Example Level Structure

```
levelX/
  ├── flag            # Password for the next level
  └── walkthrough.md  # Step-by-step solution
```

## Topics Covered

- Buffer overflows (stack/heap)
- Format string vulnerabilities
- GOT/PLT overwrites
- Shellcode injection
- Heap exploitation
- Environment variable tricks

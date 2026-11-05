---
layout: post
title: "Linux file permissions explained: rwx, owners, groups and chmod"
---

## The idea in one paragraph

Every file and folder on Linux carries a small set of rules that decide who can read it, who can change it and who can run it. These rules are the first line of defence on any Linux system. If they are too strict, legitimate users get locked out. If they are too loose, anyone on the machine can read private data or quietly replace a program with something malicious. Learning to read them takes about ten minutes, and you will use the skill every time you touch a Linux server.

## How it works

Run `ls -l` on a file and you get a line like this:

```
-rw-r--r-- 1 user user 0 Jan  1 12:00 test.txt
```

The first block, `-rw-r--r--`, is the permission string. It has ten characters:

1. **The first character is the type.** `-` means a regular file and `d` means a directory (a folder).
2. **The next nine are three groups of three:**
   - characters 2 to 4 are for the **owner** of the file
   - characters 5 to 7 are for the **group** the file belongs to
   - characters 8 to 10 are for **everyone else** (often called "others")

Inside each group of three, the letters always appear in the same order:

| Letter | Meaning for a file | Meaning for a directory |
| --- | --- | --- |
| `r` | read the contents | list what is inside |
| `w` | change the contents | create, rename or delete files inside |
| `x` | run it as a program | enter it with `cd` |

A `-` in any position means that permission is missing. So `-rw-r--r--` reads as: a regular file, the owner can read and write, the group can read, everyone else can read.

The two names after the number (`user user`) are the owner and the group. Linux checks them in order: if you are the owner, the owner permissions apply. If not, but you are in the group, the group permissions apply. Otherwise you get the "others" permissions.

### Changing permissions with numbers

The `chmod` command changes permissions, and the quickest way to use it is with three digits. Each letter has a value:

| Letter | Value |
| --- | --- |
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |

Add up the values you want for each group, and write the three totals in the order owner, group, others:

| Number | Owner | Group | Others | Result |
| --- | --- | --- | --- | --- |
| `644` | 4+2 = rw | 4 = r | 4 = r | `-rw-r--r--` |
| `600` | 4+2 = rw | 0 | 0 | `-rw-------` |
| `700` | 4+2+1 = rwx | 0 | 0 | `-rwx------` |
| `777` | rwx | rwx | rwx | `-rwxrwxrwx` |

`644` is the usual default for files. `600` is what you want for anything private, such as an SSH key.

## Try it yourself

On any Linux machine, or WSL on Windows:

```bash
touch test.txt        # create an empty file
ls -l test.txt        # see its permissions
chmod 600 test.txt    # owner only: read and write
ls -l test.txt        # check the change
chmod 644 test.txt    # back to the normal default
```

Here is what the output looks like:

```
$ ls -l test.txt
-rw-r--r-- 1 user user 0 Jan  1 12:00 test.txt
$ chmod 600 test.txt
$ ls -l test.txt
-rw------- 1 user user 0 Jan  1 12:00 test.txt
$ chmod 700 test.txt
$ ls -l test.txt
-rwx------ 1 user user 0 Jan  1 12:00 test.txt
```

## Why it matters in security

Permissions decide how far an attacker can get once they are inside a system. A few common real-world problems:

- **World-writable files (`777`).** If a script that runs automatically is writable by everyone, any user on the machine can add their own commands to it, and those commands will run with the script owner's privileges. This is a classic way to escalate from a low-privilege account to a powerful one.
- **Readable secrets.** Config files holding database passwords or API keys are often left at `644`, so every user on the server can read them. They should usually be `600`.
- **SSH keys.** SSH actually refuses to use a private key if its permissions are too open, because a key anyone can read is no longer private.

When security tools or penetration testers audit a Linux machine, one of the first things they look for is files with permissions that are looser than they need to be. Being able to read a line like `-rwxrwxrwx` at a glance is the starting point for spotting that.

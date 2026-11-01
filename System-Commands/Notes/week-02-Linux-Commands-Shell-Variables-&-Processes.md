# Week 2 — Linux Commands, Shell Variables & Processes

## Overview

Week 2 builds on Week 1 — Linux & Virtual Machines. It covers richer file listing and text-inspection commands, command discovery and aliases, links and disk usage, shell variables and expansion, and process/job control. The lecture material is dated December 22, 2021; check each command’s local manual for details that vary by shell or Linux distribution. 

## 1. `ls`: options, arguments, and recursion

`ls` lists directory contents. Options change the output; an operand (argument) tells it which file or directory to inspect. Short options commonly use one dash (`-a`); some commands also offer longer, descriptive forms (for example, GNU `ls --all`). Multiple short options can often be combined.

```bash
ls -l                 # detailed listing of the current directory
ls -l Documents        # list Documents
ls -al                 # detailed listing, including hidden entries
ls -la                 # same options in a different order
ls -R Documents        # recursively list Documents and its subdirectories
```

- `-a` includes hidden entries (names beginning with `.`); `-l` uses long format; `-R` recurses into subdirectories.
- You can give `ls` more than one path. Check `man ls` for option names, ordering, and exact behavior on your system.
- For commands that accept multiple source paths, the final argument is commonly the destination. For `cp`, copying directories requires recursive mode such as `cp -R source-dir destination`; `mv` can move directories without a recursive flag.

## 2. Read and inspect text files

| Command | Use |
| --- | --- |
| `less <file>` | Read interactively, page by page; press `q` to quit. Often preferable for long files. |
| `more <file>` | Another page-at-a-time viewer. |
| `cat <file>` | Print the whole file to the terminal; useful for short files or combining files. |
| `head <file>` | Show the first 10 lines by default; use `head -n 20 <file>` to choose a count. |
| `tail <file>` | Show the last 10 lines by default; `tail -n 20 <file>` chooses a count. |
| `wc <file>` | Count lines, words, and bytes; `wc -l <file>` reports lines only. |

## 3. Find out what commands mean and where they come from

- `which <command>` — show the executable path found for a command (where applicable).
- `type <command>` — explain how the current shell interprets a command; it may be an alias, function, builtin, or executable.
- `whatis <command>` — show a brief manual-page description.
- `apropos <keyword>` — search manual descriptions for a keyword.
- `help` or `help <builtin>` — show help for Bash builtins and shell keywords.
- `man <command>` — open a command’s manual page.
- `info` — browse GNU Info documentation; use its navigation prompts and the `q` key to quit.

When unsure, try `type`, `command --help`, or `man command`; available help varies by command.

## 4. Aliases

An **alias** is a shell nickname for a command line. In Bash, aliases are usually for convenience in interactive use.

```bash
alias ll='ls -al'   # define an alias
alias               # list aliases
unalias ll          # remove the alias
\ls                  # run ls without expanding an alias named ls
```

Aliases normally last only for the current shell unless added to a shell startup file. Review an alias before relying on it, especially if it changes or adds options.

## 5. Multiple paths, links, and file sizes

### Multiple arguments

Many commands accept a list of paths. For copy and move operations, the destination is generally the final argument; when there are multiple sources, it must be a directory. `cp` does not copy directories recursively unless asked to (commonly with `-R`); `mv` can move a directory tree as a unit.

### Hard links and symbolic links

A **symbolic link** (symlink) stores a path to another file or directory, rather like a shortcut. A **hard link** is another directory entry for the same underlying file data (same inode).

```bash
ln -s <source> <link-name>   # create a symbolic link
ln <source> <link-name>      # create a hard link
```

A symlink can point across filesystems and may become dangling if its target is moved or removed. A hard link refers to the same inode, typically must be on the same filesystem, and generally cannot be made to a directory. Removing one hard-link name does not remove the data while another hard link still refers to it.

### File and filesystem usage

- `ls -s` — show allocated block counts alongside names (not necessarily the exact logical file size).
- `stat <file>` — show file metadata, including size and timestamps.
- `du -h <path>` — estimate disk usage for a file or directory in human-readable units.
- `df -h` — show filesystem capacity and available space in human-readable units.
- `free -h` — show memory and swap statistics; this is RAM/swap, not disk space.

### Virtual system filesystems

`/proc` and `/sys` expose process, kernel, device, and system information through virtual filesystems; their contents are not ordinary files stored like documents on a disk. Inspect them cautiously. **Never run broad destructive commands such as recursive deletion against system paths or as root.**

## 6. Shell variables and `echo`

`echo` prints text and, in an unquoted or double-quoted context, the shell can expand variables written with `$`.

```bash
echo "Hello, world"
echo "$HOME"
echo "User is $USER"
echo 'User is $USER'      # single quotes preserve the literal $USER
echo "User is \$USER"   # backslash escapes the dollar sign
```

Common environment variables include `$USER` (or sometimes `$USERNAME`), `$HOME`, `$HOSTNAME`, `$PWD` (current directory), and `$PATH` (directories searched for commands). Names and availability can differ by environment.

- `printenv` or `env` — display environment variables; `printenv HOME` prints one value.
- `set` — in Bash, display or change shell options and variables and set positional parameters. Shell variables need not be exported to child processes; environment variables are exported.
- Special Bash parameters: `$0` (shell/script name), `$$` (shell process ID), `$?` (exit status of the previous command), and `$-` (current shell option flags).

## 7. Processes and job control

A process is a running program. Use `ps` (or `ps --forest` for a process tree) to inspect processes; `top` provides a live, interactive overview. In `top`, press `q` to quit.

```bash
sleep 5             # wait for five seconds in the foreground
sleep 30 &          # start a job in the background
jobs                # list jobs started by this shell
fg                  # bring the current job to the foreground
coproc sleep 20     # start a Bash coprocess asynchronously
```

- Append `&` to start a command as a background job and return to the prompt.
- `jobs` lists jobs managed by the current shell; `fg` brings a job forward (use `fg %<job-number>` to select one).
- `Ctrl + C` sends an interrupt to a foreground job. `Ctrl + Z` suspends a foreground job; use `jobs` to see it and `fg` to resume it in the foreground.
- `coproc` runs a command asynchronously in a subshell with pipes connected to the parent shell; it is more specialized than simply appending `&`.
- `kill <PID>` requests that a process stop (normally by sending `TERM`). Use `ps` to identify a PID and check it carefully before acting. `kill -9 <PID>` sends `KILL`, which cannot be caught or handled; use it only when a normal termination request fails and you are certain of the target.

## 8. Exit statuses

After a command finishes, `$?` contains its exit status. By convention, `0` means success and a nonzero value indicates some kind of failure. Common Bash-related values include:

| Status | Common meaning |
| --- | --- |
| `0` | Success |
| `1` | General error |
| `2` | Misuse of a shell builtin or command |
| `126` | Command found but cannot be executed |
| `127` | Command not found |
| `128 + N` | Common convention for termination by signal `N` |
| `130` | Often indicates interruption with `Ctrl + C` (`128 + SIGINT`) |
| `137` | Often indicates termination by `SIGKILL` (`128 + 9`) |

Exact statuses can depend on the program and shell. Bash exit statuses are represented in the range 0–255; for example, an out-of-range value such as `300` wraps to `44`.

## 9. Bash flags, history, and expansion

### Bash option flags

`echo "$-"` displays flags for the current Bash shell. Common flags include `i` (interactive), `m` (job control), `B` (brace expansion), `h` (command hashing), and `H` (history expansion). The exact set depends on how Bash was started. `bash -c 'command'` runs commands in a non-interactive Bash instance.

### Command history

- `history` — list commands recorded for the current shell.
- `!n` — run history entry number `n`.
- `!!` — repeat the previous command.

**Safety tip:** inspect the recalled command before repeating it, particularly if it deletes files, changes permissions, or affects system state.

### Brace expansion and wildcards

Brace expansion generates text patterns before a command runs:

```bash
echo a{b,c,d}e    # abe ace ade
echo {a..z}       # letters a through z
echo {A..C}{g..k} # combinations of the two ranges
```

The `*` wildcard is a **filename glob**: the shell expands it to matching names in the current directory before starting the command. For example, `D*` matches names that begin with `D`. Unlike brace expansion, a glob matches existing filesystem names. Quote a pattern when you want to pass it literally rather than have the shell expand it.

## Quick practice

1. Compare `ls -al` with `ls -R <directory>`; try supplying a directory as an argument.
2. Read a long file with `less`, then inspect its beginning, end, and counts with `head`, `tail`, and `wc`.
3. Run `type ls`, `which ls`, `whatis ls`, and `apropos directory`; compare what each tells you.
4. Set and remove a harmless alias, then use `\ls` to bypass a same-named alias.
5. In a temporary practice directory, compare a symlink and hard link with `ls -li` and `stat`.
6. Try `echo "$HOME"`, `echo '$HOME'`, `echo "$?"`, and `echo "$-"` and explain the differences.
7. Start `sleep 30 &`, inspect it with `jobs`/`ps`, and bring it to the foreground with `fg`; stop it with `Ctrl + C`.
8. Try brace expansion and a filename glob in a safe directory; check the result before using a pattern in any file-changing command.

## Lecture links

- Simple Commands in Linux — 2 [[1]](https://youtu.be/lpe6AKk5GIk)
- Shell Variables [[2]](https://youtu.be/pPRge8Yxbso)
- Linux Process Management [[3]](https://youtu.be/2aThmDRvSWU)
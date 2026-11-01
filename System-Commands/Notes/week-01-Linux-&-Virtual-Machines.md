# Week 1 — Linux & Virtual Machines


## Overview

Week 1 covers setting up Linux in a virtual machine, using the terminal, navigating the Linux filesystem, inspecting system and file information, and managing files and permissions. The three lectures are dated December 22, 2021, so software versions and interface labels may have changed since then.

## 1. Run Linux in a virtual machine

### Why use a VM?

A **virtual machine (VM)** runs a guest operating system—such as Linux—inside the host computer’s existing operating system. It lets you try Linux without replacing your current OS or setting up a dual-boot system.

A **hypervisor** (also called a virtual machine monitor or VMM) creates and runs VMs. It allocates some of the host computer’s resources, such as memory, processing capacity, and disk space, to each guest VM.

### What you need

- An operating-system installer image, usually an `.iso` file. The lecture recommends Ubuntu 20.04; choose a currently supported release when setting up a new VM.
- A hypervisor such as Oracle VirtualBox or VMware Workstation Player. Windows Subsystem for Linux (WSL) is another way to use Linux tools on Windows, though it is not the same as a traditional desktop VM.
- Disk space for the VM: the lecture recommends at least 20 GB.
- Host memory: the lecture recommends 8 GB or more of RAM overall. Allocate only a portion to the VM so the host remains usable.

### VirtualBox installation outline

1. Download the Ubuntu desktop `.iso` from ubuntu.com/download/desktop.
2. Install VirtualBox (or another hypervisor).
3. Create a new VM. Give it a name and choose the appropriate OS type and version.
4. Allocate memory and virtual disk space. Keep enough resources available for the host computer.
5. Open the VM’s settings and go to **Storage**. Select the empty optical drive, choose the virtual optical disk option, and attach the downloaded `.iso`.
6. Start the VM and follow the operating-system installer prompts.
7. After installation, restart the VM if prompted. The VM should boot into the installed guest OS.

The lecture’s screenshots use older VirtualBox and Ubuntu versions; current menus may look different.

## 2. Command-line environment

### Why use the command line?

The command line provides direct access to system tools. Commands can be combined into scripts, which makes repeated tasks easier to automate. A command usually has this general form:

```
command [options] [arguments]
```

- **Command**: the program to run, such as `ls`.
- **Option/flag**: changes how the command behaves, such as `-a`.
- **Argument**: the item the command acts on, such as a file or path.

For example, in `ls -a`, `ls` is the command and `-a` is an option. Angle-bracket text in examples—such as `<file-name>`—is a placeholder; replace it with a real name and do not type the brackets.

### Terminal basics

| Goal | Command or shortcut |
| --- | --- |
| Clear the terminal display | `clear` or `Ctrl + L` |
| Print the current directory | `pwd` |
| List directory entries | `ls` |
| Show hidden entries | `ls -a` |
| Show a detailed listing | `ls -l` |
| Show hidden entries in detailed form | `ls -al` |
| Display running processes | `ps` |
| Print system/kernel information | `uname` |
| Open a command’s manual | `man <command>` (for example, `man ls`) |
| Exit the shell | `exit` or `Ctrl + D` |

A newly opened terminal commonly starts in the current user’s home directory. Names beginning with a dot, such as `.bashrc`, are hidden in ordinary `ls` output; `ls -a` includes them. Options can often be combined, as in `ls -al`.

## 3. Linux filesystem and paths

Linux arranges files and directories in a tree that starts at the filesystem root, `/`. The slash is also the separator between directory names in a path.

- `/` — filesystem root.
- `.` — current directory.
- `..` — parent directory.
- **Absolute path** — begins at `/`, for example `/home/utkarsh/Documents`.
- **Relative path** — begins from the current directory, for example `Documents/notes.txt`.

Use `cd` (“change directory”) to move through the tree:

```bash
cd                 # go to your home directory
cd Documents       # enter a child directory named Documents
cd ..              # go to the parent directory
cd /               # go to the filesystem root
cd -               # return to the previous directory
```

### Common top-level directories

| Directory | Typical purpose |
| --- | --- |
| `/bin` | Essential command binaries. On many modern distributions, this may be linked into `/usr/bin`. |
| `/boot` | Bootloader and other boot-related files. |
| `/dev` | Device files representing devices and system interfaces. |
| `/etc` | System-wide configuration files. |
| `/home` | Home directories for regular users. |
| `/lib` | Essential shared libraries and kernel modules; often integrated with `/usr/lib` on modern systems. |
| `/media` | Common mount points for removable media. |
| `/mnt` | A mount point often used for temporary mounts. |
| `/opt` | Optional or add-on application software. |
| `/run` | Runtime data for processes since the current boot. |
| `/sbin` | Essential system-administration binaries; often integrated with `/usr/sbin` on modern systems. |
| `/srv` | Data used by services hosted by the system. |
| `/tmp` | Temporary files; contents may be removed automatically. |
| `/usr` | A major hierarchy for user-space programs, libraries, and shared data. |
| `/var` | Variable data that changes during system operation, including logs and caches. |

Important parts of the `/usr` hierarchy:

- `/usr/bin` — user commands and applications.
- `/usr/lib` — libraries used by programs.
- `/usr/local` — software and files installed locally by an administrator.
- `/usr/sbin` — system-administration programs.
- `/usr/share` — architecture-independent shared data, such as documentation.
- `/usr/include` — header files used when compiling C programs.
- `/usr/src` — source code, often including kernel source packages.

Important parts of the `/var` hierarchy:

- `/var/cache` — application cache data.
- `/var/lib` — variable state data maintained by applications and services.
- `/var/local` — variable data for locally installed software under `/usr/local`.
- `/var/lock` — lock files or lock-related data; implementation may vary by distribution.
- `/var/log` — system and application logs.
- `/var/run` — runtime data; commonly linked to `/run` on modern systems.
- `/var/tmp` — temporary files that are generally preserved longer than files in `/tmp`, including across reboots.

## 4. Everyday system and inspection commands

| Command | What it does |
| --- | --- |
| `date` | Prints the current date and time. |
| `date -R` | Prints the date in an RFC-style format. |
| `cal` | Displays a calendar, typically for the current month. |
| `free` | Shows memory and swap usage. |
| `free -h` | Shows memory and swap values in human-readable units. |
| `groups` | Lists the groups the current user belongs to. |
| `whoami` | Prints the current username. |
| `uname` | Prints basic system or kernel information. |
| `ps` | Lists processes associated with the current session by default. |
| `file <name>` | Identifies the type or format of a file. |
| `less <file>` | Reads a text file one screen at a time. |

Use `man <command>` to read a command’s manual, for example `man chmod` or `man ls`. In `less`, use the arrow keys or Page Up/Page Down to move; press `q` to quit.

## 5. Read detailed directory listings

`ls -l` displays a long listing. A typical line contains:

1. **File type and permissions**, such as `drwxr-xr-x`.
2. **Link count**.
3. **Owner**.
4. **Group**.
5. **Size** (usually bytes for ordinary files).
6. **Last-modified date/time**.
7. **Name**.

The first character indicates the object type:

| Character | Type |
| --- | --- |
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `s` | Socket |
| `p` | Named pipe (FIFO) |

An **inode** is a filesystem record containing metadata about a file or directory and information used to locate its data. A directory maps names to filesystem objects; the inode is not the filename itself.

## 6. Permissions

A permission string has a type character followed by three groups of permissions:

```
-rwxr-xr--
  owner group others
```

Each group can have:

- `r` — read.
- `w` — write.
- `x` — execute. For a directory, `x` allows traversal/search through it.
- `-` — the permission is not granted.

The groups apply to the file’s **owner**, its **group**, and **others** (everyone else), in that order. For directories, read generally allows listing names, write allows changing directory entries, and execute allows accessing entries when the path is known.

### Symbolic `chmod`

The general form is:

```
chmod <who><operator><permissions> <file-or-directory>
```

- `u` — owner/user; `g` — group; `o` — others; `a` — all three classes.
- `+` adds permission; `-` removes permission; `=` sets the permissions for that class exactly.
- `r`, `w`, and `x` represent read, write, and execute.

Examples:

```bash
chmod g-w project       # remove group write permission
chmod g-x project       # remove group execute permission
chmod g+w project       # add group write permission
chmod u+x script.sh     # add execute permission for the owner
```

### Numeric `chmod`

Numeric permissions use the values `r = 4`, `w = 2`, and `x = 1`. Add the values for each class—owner, group, others—to form a three-digit mode.

| Value | Permissions |
| --- | --- |
| `7` | `rwx` (4 + 2 + 1) |
| `6` | `rw-` (4 + 2) |
| `5` | `r-x` (4 + 1) |
| `4` | `r--` |
| `1` | `--x` |

Examples:

- `chmod 700 item` gives the owner `rwx` and gives the group and others no permissions.
- `chmod 755 item` gives the owner `rwx` and gives the group and others `r-x`.

Use permission changes carefully, especially on shared or system files. Practice on a test file or directory.

## 7. Create, copy, move, rename, and remove files

| Command | Purpose | Example |
| --- | --- | --- |
| `mkdir <name>` | Create a directory. | `mkdir linux-practice` |
| `touch <name>` | Create an empty file if it does not exist; otherwise update its timestamp. | `touch notes.txt` |
| `cp <source> <destination>` | Copy a file to another name or path. | `cp notes.txt notes-copy.txt` |
| `mv <source> <destination>` | Move or rename a file. | `mv notes.txt archive/notes.txt` or `mv draft.txt final.txt` |
| `rm <name>` | Remove a file. | `rm old-notes.txt` |

If a filename contains spaces, quote it so the shell treats it as one argument:

```bash
touch "first note.txt"
mv "draft notes.txt" "final notes.txt"
```

**Important:** `rm` normally removes the target without asking for confirmation. Use `rm -i <file>` to request an interactive confirmation, and check the path carefully before removing anything.

### Aliases

An alias assigns a short name to a longer command. For example:

```bash
alias ll="ls -altrhF"
```

This makes `ll` run `ls` with the listed options in the current shell session. To make an alias available in future shell sessions, add it to the appropriate shell startup file, such as `~/.bashrc`, then reload that file or open a new terminal.

## 8. Practice checklist

1. Run `pwd` and `ls -al` to inspect your current location and its contents.
2. Create a practice directory with `mkdir linux-practice`, then enter it with `cd linux-practice`.
3. Create a file with `touch "first note.txt"`; inspect it using `ls -l` and `file "first note.txt"`.
4. Try `cd ..`, `cd -`, and `cd /`; use `pwd` to confirm where you are.
5. Copy and rename the practice file with `cp` and `mv`.
6. Try permission changes only on the practice file or directory, then inspect the result with `ls -l`.
7. Use `man chmod` or `man ls` to look up options. Remove practice files only after checking the target.

## Lecture links

- Launching a Virtual Machine  [[1]](https://youtu.be/PHrN7yp1AJw)
- Command Line Environment     [[2]](https://youtu.be/qrAnlpcMyYc)
- Simple Commands in Linux — 1 [[3]](https://youtu.be/DIpBEmRDHnw)
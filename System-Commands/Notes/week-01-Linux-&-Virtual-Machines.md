# Week 1 — Linux & Virtual Machines


## Overview

Week 1 introduces running Linux safely in a virtual machine, using the command line, navigating the Linux filesystem, and managing files and permissions. The lecture material is dated December 22, 2021; specific OS and hypervisor versions may have changed since then.  

## 1. Run Linux in a virtual machine

A **virtual machine (VM)** lets you try Linux as a guest operating system without replacing your computer’s current OS or setting up dual boot. A **hypervisor** (or virtual machine monitor) creates and runs VMs by allocating host resources such as memory and processing.

### What you need

- An operating-system `.iso` installer image (the lecture recommends Ubuntu 20.04).
- A hypervisor such as VirtualBox or VMware Workstation Player; Windows Subsystem for Linux is another option mentioned in the lecture.
- At least 20 GB of free disk space for the VM; the lecture recommends 8 GB or more of host RAM.

### Basic VirtualBox workflow

1. Download the Ubuntu desktop ISO and install VirtualBox.
2. Create a new VM; choose a name, Linux type, and Ubuntu version.
3. Allocate memory and storage while leaving enough resources for the host.
4. Open the VM’s **Settings → Storage**, select the empty optical drive, and attach the ISO.
5. Start the VM and follow the installer prompts.

## 2. Command-line essentials

A shell command usually consists of a command followed by optional options (flags) and arguments. For example, in `ls -a`, `ls` is the command and `-a` is an option.

| Goal | Command / shortcut |
| --- | --- |
| Clear the terminal | `clear` or `Ctrl + L` |
| Show the current directory | `pwd` |
| List files and folders | `ls` |
| Include hidden entries | `ls -a` |
| Show a detailed listing | `ls -l` |
| Combine listing options | `ls -al` |
| Show running processes | `ps` |
| Show system/kernel name | `uname` |
| Open a command manual | `man <command>` (example: `man ls`) |
| Exit the shell | `exit` or `Ctrl + D` |

By default, a new terminal typically starts in the current user’s home directory. Hidden names generally begin with a dot, so `ls -a` reveals them.

## 3. Navigate the filesystem

Linux organizes files as a directory tree rooted at `/`.

- `/` — filesystem root; also separates directory names in a path.
- `.` — current directory.
- `..` — parent directory.
- **Absolute path** — starts at `/`; **relative path** — starts from the current directory.

```bash
cd                 # go to your home directory
cd <folder-name>   # enter a child directory
cd ..              # go to the parent directory
cd /               # go to the filesystem root
cd -               # return to the previous directory
```

### Common top-level directories

- `/bin` — essential command binaries.
- `/boot` — bootloader and boot-related files.
- `/dev` — device files.
- `/etc` — system configuration.
- `/home` — users’ home directories.
- `/lib` — essential shared libraries and kernel modules.
- `/media` — mount points for removable media.
- `/mnt` — mount points, often used temporarily.
- `/opt` — add-on application software.
- `/run` — runtime data for processes.
- `/sbin` — essential system binaries.
- `/srv` — data for services.
- `/tmp` — temporary files.
- `/usr` — much of the user-space software hierarchy, including `/usr/bin`, `/usr/lib`, `/usr/local`, `/usr/share`, `/usr/include`, and `/usr/src`.
- `/var` — variable data, including logs (`/var/log`), caches (`/var/cache`), state (`/var/lib`), and temporary data (`/var/tmp`).

## 4. Useful system and file commands

| Command | What it does |
| --- | --- |
| `date` / `date -R` | Show date and time; `-R` uses an RFC-style date format. |
| `cal` | Display a calendar. |
| `free -h` | Show memory and swap usage in human-readable units. |
| `groups` | List the groups the current user belongs to. |
| `whoami` | Print the current username. |
| `file <name>` | Identify a file’s type. |
| `less <file>` | Read a text file page by page. |
| `mkdir <name>` | Create a directory. |
| `touch <name>` | Create an empty file if it does not exist, or update its timestamp. |
| `cp <source> <destination>` | Copy a file. |
| `mv <source> <destination>` | Move or rename a file. |
| `rm <file>` | Remove a file. |

Quote filenames that contain spaces, for example: `mv "draft notes.txt" "final notes.txt"`.

> **Caution:** `rm` normally deletes without asking for confirmation. Check the target carefully; use `rm -i <file>` for an interactive prompt.
> 

An alias can shorten a frequently used command:

```bash
alias ll="ls -altrhF"
```

## 5. Read `ls -l` output and understand permissions

A long listing includes the file type and permissions, link count, owner, group, size, modification time, and name. The first character indicates the type:

- `-` regular file; `d` directory; `l` symbolic link.
- `c` character device; `b` block device; `s` socket; `p` named pipe.

An **inode** is filesystem metadata associated with an object. `ls -l <name>` displays a permissions string such as `drwxr-xr-x`: the first character is the type, followed by permissions for **owner**, **group**, and **others**. Each group of three uses `r` (read), `w` (write), and `x` (execute).

### Change permissions symbolically

```bash
chmod g-w <name>   # remove group write permission
chmod g-x <name>   # remove group execute permission
chmod g+w <name>   # add group write permission
```

In symbolic mode, `u` means owner/user, `g` group, and `o` others; `+` adds and `-` removes permissions. For example, `chmod u+x script.sh` adds execute permission for the owner.

### Change permissions numerically

The values are `r = 4`, `w = 2`, and `x = 1`; add them for each class (owner, group, others). For example, `chmod 700 <name>` gives the owner `rwx` and gives group and others no permissions. `755` means owner `rwx`, group `r-x`, and others `r-x`.

## Quick practice

1. Run `pwd` and `ls -al` to inspect your current location.
2. Create a practice directory with `mkdir linux-practice`, then enter it with `cd linux-practice`.
3. Create a file using `touch "first note.txt"`; inspect it with `ls -l` and `file "first note.txt"`.
4. Try `cd ..`, then `cd -`, and use `man chmod` to explore the manual.
5. Practice permission changes only on your test file or directory.

## Lecture links

- Launching a Virtual Machine  [[1]](https://youtu.be/PHrN7yp1AJw)
- Command Line Environment     [[2]](https://youtu.be/qrAnlpcMyYc)
- Simple Commands in Linux — 1 [[3]](https://youtu.be/DIpBEmRDHnw)
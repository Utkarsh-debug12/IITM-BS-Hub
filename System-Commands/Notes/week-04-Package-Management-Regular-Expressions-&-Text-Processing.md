# Week 4 — Package Management, Regular Expressions & Text Processing

## Overview

Week 4 covers Linux package management (package formats, architectures, repositories, checksums, `apt`, and `dpkg`) and text pattern matching with regular expressions, `grep`, and `cut`. The examples focus on Ubuntu/Debian and GNU tools; flags and behavior can differ across distributions. 

## 1. Why package managers?

A package manager installs, updates, removes, and queries software. It typically:

- Retrieves packages from configured repositories over a network.
- Tracks installed packages and versions in a local package database.
- Resolves dependencies so required supporting packages are installed.
- Verifies package integrity/authenticity using repository metadata and signatures.
- Provides tools and metadata for building packages.

Prefer your distribution’s package manager and trusted repositories over manually downloading packages. Package managers understand dependencies and system state; installing a standalone package file may not resolve dependencies automatically.

## 2. Package formats, distributions, and architecture

Linux distributions use different package ecosystems. Debian-based distributions such as Debian, Ubuntu, and Mint use `.deb` packages; Red Hat-family distributions commonly use RPM packages. Tools are ecosystem-specific: `apt` and `dpkg` manage Debian packages, while `dnf`/`yum` and `rpm` are associated with RPM systems.

Typical package filenames include package name, version, release/revision, and architecture. For example, RPM filenames commonly follow `name-version-release.architecture.rpm`; Debian filenames commonly follow `name_version-revision_architecture.deb`.

Architecture labels identify the target CPU/platform. Common examples include `amd64`/`x86_64` (64-bit x86), `i386`/`x86` (32-bit x86), `arm`, and `ppc64el` (little-endian 64-bit PowerPC). `all`, `noarch`, or `src` may indicate architecture-independent, generic, or source packages depending on the format.

```bash
uname -a             # kernel and machine details
uname -m             # machine hardware name / architecture
lsb_release -a       # distribution information, if installed
```

Choose packages compatible with both your distribution/release and architecture.

## 3. Inspect Ubuntu package information with `apt-cache`

`apt-cache` searches and displays information from the local APT package metadata. Refresh that metadata with `sudo apt update` before relying on search results.

```bash
apt-cache search keyword          # search package names/descriptions
apt-cache pkgnames                # list package names known to APT
apt-cache pkgnames prefix         # list names beginning with a prefix (where supported)
apt-cache show package-name       # show package record(s)
```

A package record includes fields such as version, dependencies, description, and repository details. Package names and metadata vary by Ubuntu release. Package categories/sections (for example, `math`, `editors`, or `web`) help classify packages; Ubuntu’s package index can be browsed at packages.ubuntu.com.

Some Debian/Ubuntu package metadata uses priority labels such as `required`, `important`, `standard`, `optional`, and `extra`. These describe a package’s role in a particular package set; do not treat them as a complete measure of software quality or security.

## 4. Checksums and package trust

A checksum is a digest calculated from file contents. If a file changes, its digest will usually change. Compare a downloaded file’s checksum with one obtained through a trusted, independent source:

```bash
md5sum file
sha1sum file
sha256sum file
```

MD5 and SHA-1 are obsolete for security-sensitive integrity verification because of collision weaknesses. Prefer SHA-256 or stronger when a checksum is needed. A checksum alone does not prove who supplied a file: use trusted repository signatures and verified distribution channels for authenticity.

## 5. APT: update, install, upgrade, and remove

Administrative package changes usually require `sudo` privileges. APT repositories are configured under `/etc/apt/`; `sources.list` contains repository entries and `sources.list.d/` holds additional source-list files. Only add repositories you trust.

A common maintenance sequence is:

```bash
sudo apt update            # refresh package-index metadata
sudo apt upgrade           # upgrade installed packages where possible
sudo apt autoremove        # review/remove unneeded automatically installed dependencies
```

`apt update` refreshes available-package information; it does not itself upgrade installed software. Read the proposed changes before confirming. `-y` automatically answers yes, so use it only when you have reviewed and intend to approve the operation.

```bash
sudo apt install package-name       # install a package
sudo apt reinstall package-name    # reinstall an installed package
sudo apt remove package-name       # remove package, generally keep system config files
sudo apt purge package-name        # remove package and its system configuration files
sudo apt clean                     # clear downloaded package archives from APT cache
sudo apt autoremove                # remove eligible unused dependencies
```

`remove` versus `purge`: purge also removes package-managed configuration files. Review APT’s summary before proceeding, especially when removing or purging packages.

## 6. `sudo`, permissions, and logs

The `sudo` policy is configured through `/etc/sudoers` and related configuration. Use `sudo` for authorized administrative tasks rather than editing package files directly. View sudo policy only if you have permission:

```bash
sudo less /etc/sudoers
```

Do not edit `/etc/sudoers` with a regular text editor; administrators should use `visudo`, which checks syntax before saving. Failed or successful authentication/sudo events may be logged under `/var/log/` (for example, `auth.log` on some Ubuntu systems); exact log paths and access depend on the distribution and configuration.

## 7. Query and install Debian packages with `dpkg`

`dpkg` is the lower-level Debian package tool. Its database is stored under `/var/lib/dpkg/`; files such as `status` track package states, while `info/` contains package-specific metadata.

```bash
dpkg -l 'pattern'                # list packages matching a package-name pattern
dpkg -L package-name             # list files installed by a package
dpkg -s package-name             # show installed-package status/details
dpkg -S '/path/to/file'          # find installed package owning a file path
dpkg-query -W                    # query package database
```

Example: print each installed package’s section and binary package name:

```bash
dpkg-query -W -f='${Section} ${binary:Package}\n'
```

Install a local Debian archive with:

```bash
sudo dpkg -i ./package_file.deb
```

`dpkg -i` installs a local file but does not automatically fetch/resolve all dependencies. For routine installation, use APT and trusted repositories; if a local `.deb` is necessary, APT can often resolve its dependencies with `sudo apt install ./package_file.deb`. Prefer APT for removal as well, because it handles dependency planning.

## 8. Regular expressions and `grep`

A regular expression (regex) is a pattern used to match text. Regexes are used in tools such as `grep`, `sed`, and `awk`, programming languages, and database systems.

```bash
grep 'pattern' filename       # print lines matching pattern
grep 'pattern' < filename     # read input from a file
grep 'pattern' filename       # equivalent direct file search
command | grep 'pattern'      # filter output from another command
```

By default, `grep` uses POSIX Basic Regular Expressions (BRE). Use Extended Regular Expressions (ERE) with `grep -E`; `egrep` is a legacy synonym.

```bash
grep 'BRE-pattern' file
grep -E 'ERE-pattern' file
```

Quote patterns so the shell does not interpret metacharacters before `grep` receives them. Single quotes are usually a good choice.

### Common BRE and ERE operators

| Pattern | Meaning |
| --- | --- |
| `.` | Any one character (except newline). |
| `*` | Zero or more repetitions of the preceding character/expression. |
| `[abc]` | One character from the listed set. |
| `[a-z]` | One character in a range (interpretation can depend on locale). |
| `[^abc]` | One character not in the set; `^` negates when first inside the brackets. |
| `^` | Beginning of line anchor. |
| `$` | End of line anchor. |
| `\` | Escape a metacharacter to match it literally. |

ERE adds these convenient operators:

| ERE pattern | Meaning |
| --- | --- |
| `+` | One or more repetitions. |
| `?` | Zero or one repetition. |
| ` | ` |
| `( ... )` | Group an expression. |
| `{n,m}` | Repeat the preceding expression between `n` and `m` times. |

In BRE, interval repetition and grouping are escaped: `\{n,m\}` and `\( ... \)`. GNU `grep` also accepts some extensions, such as `\b` for a word boundary; these are not part of portable POSIX BRE/ERE. Check `man grep` for the implementation you are using.

### Character classes

POSIX character classes are written inside brackets:

| Class | Matches |
| --- | --- |
| `[[:alnum:]]` | Letters or digits. |
| `[[:alpha:]]` | Letters. |
| `[[:digit:]]` | Decimal digits. |
| `[[:lower:]]` / `[[:upper:]]` | Lowercase / uppercase letters. |
| `[[:space:]]` | Whitespace. |
| `[[:blank:]]` | Space or tab. |
| `[[:punct:]]` | Punctuation. |
| `[[:xdigit:]]` | Hexadecimal digits. |

For example, `[[:digit:]]{6}` in ERE matches a run of at least six digits if followed by a repetition operator? More precisely, the interval `{6}` matches exactly six consecutive digits, but it may still match a six-digit substring inside a longer number. Add boundaries or explicit non-digit conditions when you need the entire number to be exactly six digits.

### Anchors, literal characters, and useful flags

```bash
grep -E '^Error' file          # line begins with Error
grep -E 'failed$' file         # line ends with failed
grep -E 'a.m' file             # a, any character, then m
grep -E 'a\.m' file           # literal dot between a and m
grep -i 'warning' file         # ignore letter case
grep -v '^$' file              # exclude empty lines
```

`^` and `$` anchor a pattern to a whole line. GNU grep’s `\b` marks a word boundary; it can match a word inside a line, unlike `$`, which means end of line. Word-boundary syntax varies across tools, so for portable expressions use explicit boundaries appropriate to the data.

Within a bracket expression, `[mn]` matches either `m` or `n`; `[1-4]` matches one digit in that range; `[^0-9]` matches a character other than an ASCII digit. Examples:

```bash
grep -E 'S.*[mn]' file        # S followed later by m or n
grep -E '[[:alpha:]]{2}[[:digit:]]{2}' file
```

### Backreferences and grouping

A parenthesized subexpression can be captured and referenced later as `\1` through `\9` (support depends on the regex engine). With GNU grep BRE:

```bash
grep '\(hello\).*\1' file   # line containing hello twice, with text between
```

For ERE, grouping is unescaped, while backreference syntax remains `\1` in common grep implementations:

```bash
grep -E '(hello).*\1' file
```

Backreferences match the same text captured earlier; they are useful for repeated words or patterns. Regex operator precedence matters: repetition applies to the preceding atom/group; concatenation binds more tightly than alternation. Use parentheses to make intent clear, for example `^(cat|dog)$` matches exactly `cat` or `dog` in ERE.

## 9. Regex examples for filtering package data and text

Combine `dpkg-query` output with `grep -E` to filter records. Here, the output format is `section package-name`:

```bash
# Package names exactly 4 characters long (after the separating space)
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep -E ' .{4}$'

# Names starting with g and exactly 3 more characters
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep -E ' g.{3}$'

# Names starting with g and 1–5 additional characters
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep -E ' g.{1,5}$'

# Records whose section begins with math
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep -E '^math'

# Records whose package name contains kd
# (the exact package naming convention varies)
dpkg-query -W -f='${Section} ${binary:Package}\n' | grep -E ' kd.*$'
```

Other examples, using GNU grep/ERE:

```bash
grep -E '[[:digit:]]{12,}' file.txt                 # a run of 12 or more digits
grep -E '(^|[^[:digit:]])[[:digit:]]{6}([^[:digit:]]|$)' file.txt  # exactly 6 digits as a number
grep -E '\b[[:alpha:]]{2}[[:digit:]]{2}[[:alpha:]][[:digit:]]{3}\b' file.txt # e.g. MM22B001
grep -E '[[:alnum:]]+\.[[:alnum:]]+' file.txt       # simple domain-like text
```

The roll-number and URL patterns are teaching examples, not complete validators. Real URLs and identifiers can contain additional valid characters and edge cases. Also, `\b` is a GNU-style extension; use a portable boundary expression where needed.

## 10. Extract characters and fields with `cut`

`cut` selects horizontal portions of each input line: character positions or delimiter-separated fields. It reads from files or standard input.

### Character positions

```bash
cut -c 1-4 fields.txt     # characters 1 through 4
cut -c 5-8 fields.txt     # characters 5 through 8
cut -c -4 fields.txt      # first four characters
cut -c 8- fields.txt      # character 8 through end
```

Positions are counted from 1. `-c` selects characters (in the active locale); `-b` selects bytes.

### Delimited fields

Use `-d` to specify the delimiter and `-f` to choose fields. Field numbering starts at 1.

```bash
cut -d ' ' -f 1 fields.txt       # first space-delimited field
cut -d ' ' -f 1-2 fields.txt     # fields 1 and 2
cut -d ';' -f 1 fields.txt       # field before semicolon
cut -d ';' -f 2 fields.txt       # field after semicolon
```

`cut` uses a single delimiter character; repeated spaces are not automatically treated as one delimiter. For arbitrary whitespace-separated fields, consider `awk` instead. When input is already a file, pass its name directly rather than using `cat file | ...` (the latter is a useless-use-of-cat pattern).

For sample lines such as `234567;welcome cmdline,line-2`, extract the text between `;` and `,` with two cuts, then select the second record:

```bash
cut -d ';' -f 2 fields.txt | cut -d ',' -f 1 | sed -n '2p'
```

The lecture also demonstrates `head`/`tail` for selecting a line:

```bash
cut -d ';' -f 2 fields.txt | cut -d ',' -f 1 | head -n 2 | tail -n 1
```

## Quick review

- `apt` manages repository-based package installs and upgrades; `dpkg` works directly with Debian package files and the local package database.
- Refresh package metadata with `sudo apt update` before upgrading; inspect proposed changes before confirming.
- Check architecture and distribution compatibility before installing a package.
- Prefer signed trusted repositories; use SHA-256 rather than MD5/SHA-1 for security-sensitive checksum comparisons.
- `grep` prints matching lines; default syntax is BRE, while `grep -E` enables ERE operators such as `+`, `?`, `|`, and unescaped grouping.
- Use `^` and `$` for line boundaries, brackets for character sets, and repetition operators for counts.
- `cut -c` selects positions; `cut -d ... -f ...` selects delimiter-separated fields.

## Lecture links

- Software Management — part 1 [[1]](https://youtu.be/hG-bxCmfArc)
- Software Management — part 2 [[2]](https://youtu.be/ctn_s7SDTV0)
- Pattern Matching — part 1 [[3]] (https://youtu.be/1y85iTaqq8Y)
- Pattern Matching — part 2 [[4]] (https://youtu.be/XQUJPRc-7zA)
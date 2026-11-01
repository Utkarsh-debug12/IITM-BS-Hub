# Week 3 — Command Composition, Redirection & Bash Variables

## Overview

Week 3 covers chaining commands, running subshells, standard input/output/error, redirection and pipelines, comparing files, and Bash variables—from exporting and testing variables to string operations, attributes, and arrays. These notes follow the Week 3 lecture material; Bash-specific syntax may not work unchanged in other shells. 

## 1. Combine commands

Use these operators to control how commands run:

```bash
command1; command2       # run command2 after command1, regardless of its status
command1 && command2     # run command2 only if command1 succeeds
command1 || command2     # run command2 only if command1 fails
```

For chains, `&&` continues only while each preceding command succeeds; `||` continues only while each preceding command fails. A command’s success is usually indicated by exit status `0`; nonzero means failure.

### Run a subshell

Parentheses run a command or grouped list in a subshell:

```bash
(command1; command2)
```

The subshell has its own process context. Changes such as `cd` or variable assignments made inside it generally do not change the parent shell. Subshells can be nested.

## 2. Standard streams and file descriptors

Commands normally use three standard streams:

| Stream | File descriptor | Default |
| --- | --- | --- |
| Standard input (`stdin`) | `0` | Keyboard / terminal input |
| Standard output (`stdout`) | `1` | Terminal output |
| Standard error (`stderr`) | `2` | Terminal diagnostics |

Redirection changes where a stream comes from or goes. **Be careful:** `>` creates a file or truncates (overwrites) its existing contents; `>>` appends instead.

```bash
command > output.txt        # send stdout to a file; overwrite it
command >> output.txt       # send stdout to a file; append
command < input.txt         # read stdin from a file
command 2> errors.txt       # send stderr to a file
command > output.txt 2> errors.txt  # send stdout and stderr to separate files
command > all.txt 2>&1      # send stdout to all.txt, then send stderr to stdout
```

The order in `> all.txt 2>&1` matters: `2>&1` makes stderr go wherever stdout is going at that point. Reversing the redirections can send stderr somewhere different.

### Type text into a file with `cat`

```bash
cat > notes.txt       # type lines; Ctrl + D ends input and saves
cat >> notes.txt      # append typed lines; Ctrl + D ends input
```

`cat` reads from standard input when no input filename is provided. The first form overwrites an existing file; use the append form if you want to preserve its contents.

## 3. Pipes, `/dev/null`, and `tee`

A pipe sends the **stdout** of one command to the **stdin** of the next. It does not send stderr through the pipe by default.

```bash
ls /usr/bin | less             # browse a long listing
ls /usr/bin | wc -l            # count listing lines
command1 | command2 > result.txt  # pipe into command2; save its stdout
```

`/dev/null` discards anything written to it. Use it only when you intentionally do not need that output:

```bash
command > result.txt 2> /dev/null  # save stdout; discard stderr
```

`tee` writes piped input both to the terminal and to one or more files:

```bash
command | tee output.txt          # create/overwrite output.txt and display output
command | tee -a output.txt       # append instead
command | tee first.txt second.txt
```

## 4. Compare files with `diff`

`diff` compares text files line by line and reports differences:

```bash
diff file1.txt file2.txt
```

No output commonly means the files match. Use `man diff` to explore output formats and options.

## 5. Create, export, use, and remove Bash variables

A variable assignment has **no spaces** around `=`. Variable names conventionally begin with a letter or underscore and contain letters, digits, or underscores.

```bash
myvar="value string"
echo "$myvar"
echo "${myvar}"     # braces make the variable boundary explicit
export myvar         # make it available to child processes
```

You can also assign and export in one step: `export myvar="value string"`. A shell variable is local to that shell unless exported; exported environment variables are inherited by child processes.

```bash
unset myvar   # remove the variable
myvar=        # keep it set, but assign an empty value
```

To check whether a Bash variable is set:

```bash
[[ -v myvar ]]   # true if set, even if its value is empty
```

To test whether it is unset:

```bash
[[ -z ${myvar+x} ]]   # true only when myvar is unset
```

After a test, `$?` is `0` when the test succeeds and `1` when it fails. In scripts, prefer using the test directly in an `if` statement rather than checking `$?` separately.

## 6. Bash parameter expansion and string operations

Braces support operations on a variable without changing its value unless an assignment form is explicitly used.

### Default and alternate values

```bash
echo "${myvar:-default}"  # use default if myvar is unset or empty
echo "${myvar-default}"   # use default only if myvar is unset
echo "${myvar:+alternate}" # use alternate if myvar is set and nonempty
echo "${myvar+alternate}"  # use alternate if myvar is set, even if empty
```

These forms produce substituted text; `:+` does **not** assign or reset the variable. To assign a default when unset or empty, use `${myvar:=default}`; use `${myvar=default}` to assign only when unset. Quote expansions such as `"${myvar:-default}"` to protect spaces and prevent unintended word splitting or filename expansion.

### Names, length, and substring

```bash
echo "${!H*}"       # names of variables beginning with H
echo "${#myvar}"     # length of the string
echo "${myvar:5:4}"  # substring: start at offset 5, take 4 characters
```

Substring offsets are zero-based in Bash. If `myvar` is unset or empty, its length is zero.

### Remove a matching prefix or suffix

The `pattern` here is a Bash glob pattern, not a regular expression:

```bash
${myvar#pattern}    # remove the shortest matching prefix
${myvar##pattern}   # remove the longest matching prefix
${myvar%pattern}    # remove the shortest matching suffix
${myvar%%pattern}   # remove the longest matching suffix
```

### Replace a matching pattern

```bash
${myvar/pattern/replacement}    # replace the first match
${myvar//pattern/replacement}   # replace all matches
${myvar/#pattern/replacement}   # replace only at the beginning
${myvar/%pattern/replacement}   # replace only at the end
```

### Change letter case

```bash
${myvar,}    # lowercase the first character
${myvar,,}   # lowercase all characters
${myvar^}    # uppercase the first character
${myvar^^}   # uppercase all characters
```

Case-conversion operators are Bash features; availability can depend on the Bash version.

## 7. Variable attributes with `declare`

Bash `declare` can give a variable an attribute:

| Command | Effect |
| --- | --- |
| `declare -i myvar` | Treat assignments as arithmetic expressions. |
| `declare -l myvar` | Convert assigned values to lowercase. |
| `declare -u myvar` | Convert assigned values to uppercase. |
| `declare -r myvar` | Make the variable readonly. |

For many attributes, `declare +attribute` removes the attribute (for example, `declare +i myvar`). A readonly variable cannot be made writable again in the current shell; `declare +r` does not undo readonly status.

## 8. Bash arrays

### Indexed arrays

Indexed arrays use integer indices, starting at `0`:

```bash
declare -a arr
arr[0]="first value"
arr[1]="second value"
echo "${arr[0]}"       # one element
echo "${#arr[@]}"      # number of elements
echo "${!arr[@]}"      # indices currently in use
echo "${arr[@]}"       # all values
arr+=("another value") # append an element
unset 'arr[1]'         # remove an element
```

When passing all elements as separate arguments, use `"${arr[@]}"`; this preserves elements that contain spaces. Array indices need not remain consecutive after an element is removed.

### Associative arrays

Associative arrays map string keys to values (similar to a hash map). They require Bash:

```bash
declare -A hash
hash["a"]="value"
echo "${hash["a"]}"     # value at key a
echo "${#hash[@]}"      # number of entries
echo "${!hash[@]}"      # keys
 echo "${hash[@]}"      # values
unset 'hash["a"]'       # remove key a
```

## Quick practice

1. Compare `;`, `&&`, and `||` using harmless commands such as `true` and `false`.
2. Run `printf 'hello\n' > sample.txt`, then append a line with `>>`; inspect the file to see the difference.
3. Try `ls /usr/bin | less` and `ls /usr/bin | wc -l` to see a pipe in action.
4. Run a command with separate stdout and stderr files, then combine them with `> all.txt 2>&1`.
5. Use `tee` to save output while still seeing it, and `diff` to compare two small text files.
6. Set `myvar="hello world"`; test the default-value and string-length expansions, then remove it with `unset`.
7. Create a small indexed array and associative array; print their values, keys, and counts.

## Lecture links

- Combining commands and files [[1]](https://youtu.be/Lcx9UsS7y8Y)
- Redirections [[2]](https://youtu.be/BBh69kH_G_Y)
- Shell variables [[3]](https://youtu.be/QX5XElFRpck)
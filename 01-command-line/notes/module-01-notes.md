# Module 1 — Command Line Foundations

## Module Objective

Develop practical Linux command-line skills and learn how small Linux utilities can be combined to investigate and administer a system.

---

## 1. Filesystem Navigation

Important commands:

```bash
pwd
ls
ls -l
ls -la
cd

## 2. Creating Files and Directories

mkdir test
touch file1.txt
touch file{1..5}.txt

echo "Linux is fun" > file1.txt

cat file1.txt

---

## 3. Shell Expansion

Brace expansion lets the shell generate a sequence of names:

```bash
touch file{1..5}.txt
```

Wildcards allow us to match filenames:

```bash
ls file?.txt
ls file*.txt
```

- `?` matches exactly one character.
- `*` matches zero or more characters.
- `{1..5}` is brace expansion and generates a sequence.

One important lesson was that commas do not separate shell arguments. For example:

```bash
touch file1.txt, file2.txt
```

can create filenames containing commas. Normal shell arguments are separated by whitespace.

---

## 4. Redirection and Pipes

Overwrite a file with standard output:

```bash
echo "Linux is fun" > file1.txt
```

Append instead of overwrite:

```bash
echo "Another line" >> file1.txt
```

A pipe sends the output of one command to the input of another:

```bash
cat file1.txt | wc -l
```

Linux programs conventionally use three standard streams:

- `0` — stdin (standard input)
- `1` — stdout (standard output)
- `2` — stderr (standard error)

Errors can be discarded with:

```bash
2>/dev/null
```

For example:

```bash
ls ~/Linux-Learning /does-not-exist 2>/dev/null
```

`/dev/null` acts as a discard destination for data we do not want.

---

## 5. Learning to Help Yourself

A Linux administrator cannot memorise every command and option. An important skill is knowing how to discover information from the system itself.

### Manual pages

```bash
man ls
man find
```

For a shorter summary of a command's options:

```bash
sort --help
find --help
```

### Finding commands

```bash
which git
whereis ls
type cd
type -a echo
```

During the exercises we discovered that commands can be implemented in different ways:

- `cd` is a shell builtin.
- `pwd` is available as a shell builtin.
- `echo` is available as a shell builtin and also as an external command.
- `ls` is aliased to `ls --color=auto` on this system.
- `git` is an external executable at `/usr/bin/git`.
- `git_branch` is a shell function used by the customised prompt.

The `type` command is particularly useful because it tells us how the shell will interpret a command.

### Command History

```bash
history
history | grep find
```

Useful keyboard shortcuts:

- `Ctrl+R` — search backwards through command history.
- `Tab` — command and pathname completion.
- `Ctrl+L` — clear/redraw the terminal screen.

---

## 6. grep — Searching Inside Files

`grep` searches text for matching patterns.

Search for the word `Linux`:

```bash
grep Linux file1.txt
```

Ignore upper/lower case:

```bash
grep -i linux file1.txt
```

Ignore case and display line numbers:

```bash
grep -in linux file1.txt
```

Search recursively:

```bash
grep -r Linux ~/Linux-Learning/
```

Search recursively, ignore case, show line numbers and restrict the search to `.txt` files:

```bash
grep -rin --include="*.txt" Linux ~/Linux-Learning/
```

Important options:

- `-r` — recursive search.
- `-i` — ignore case.
- `-n` — display matching line numbers.
- `--include="*.txt"` — restrict the search to matching filenames.

An important distinction is:

> `find` locates files and directories.  
> `grep` searches for text inside files.

---

## 7. find — Locating Files and Directories

`find` searches the filesystem for files and directories matching specified conditions.

Basic examples:

```bash
find . -name "file1.txt"
find ~ -name "file1.txt"
```

Search specifically for regular files:

```bash
find ~/Linux-Learning -type f -name "*.txt"
```

Search specifically for directories:

```bash
find ~ -type d -name ".git" 2>/dev/null
```

Useful tests:

- `-type f` — regular files.
- `-type d` — directories.
- `-name` — match a filename.

### Finding Files by Size

We created test files of different sizes:

```bash
truncate -s 1K small.dat
truncate -s 1M medium.dat
truncate -s 10M large.dat
```

Then searched for files larger than 5 MB:

```bash
find . -type f -size +5M
```

A more realistic investigation searched the home directory for files larger than 100 MB:

```bash
find ~ -type f -size +100M 2>/dev/null
```

We can ask `find` to run another command on the results:

```bash
find ~ -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

Here:

- `-exec` tells `find` to execute another command.
- `{}` represents the pathname found by `find`.
- `\;` terminates the `-exec` expression.

---

## 8. Disk Usage and Sorting

To investigate disk usage we used:

```bash
du -ah ~/Videos | sort -hr | head -10
```

This is a pipeline of three separate utilities:

```text
du
 |
 v
sort
 |
 v
head
```

`du -ah` displays disk usage for files and directories in human-readable form.

`sort -hr` means:

- `-h` — understand human-readable sizes such as K, M and G.
- `-r` — reverse the ordering.

`head -10` then selects the first ten results.

This demonstrates an important Unix/Linux principle: small tools can be connected together to perform a larger task.

---

## 9. Searching by Modification Time

`find` can locate files according to when their contents were modified.

For example:

```bash
find . -type f -mtime -7
```

`-mtime` works in completed 24-hour periods.

Therefore:

```bash
find . -type f -mtime -1
```

means approximately the previous 24 hours, rather than strictly "since midnight today".

For more precise investigations we can use minutes:

```bash
find ~ -type f -mmin -60 2>/dev/null
```

This searches for regular files modified within approximately the last 60 minutes.

---

## 10. Using find -exec

We learned that there are two important ways to terminate a `find -exec` expression.

Run the command once for each result:

```bash
find ... -exec command {} \;
```

Pass multiple results to the command together:

```bash
find ... -exec command {} +
```

Therefore:

```text
{} \;    execute the command separately for each result

{} +     pass multiple results to the command in batches
```

The `+` form can be considerably more efficient when the receiving command accepts multiple filenames.

It also matters when the receiving command needs several files at once. For example, `ls -t` cannot meaningfully compare all results if `ls` is invoked separately for every individual file.

---

## 11. find -printf

Rather than passing every result to `ls`, `find` can format its own output using `-printf`.

Print the pathname:

```bash
find ~/.config -type f -printf '%p\n'
```

Useful format codes include:

- `%p` — pathname.
- `%T@` — numeric modification timestamp.
- `%TY` — year.
- `%Tm` — month.
- `%Td` — day.
- `%TH` — hour.
- `%TM` — minute.
- `\n` — newline.

A machine-friendly timestamp can be used for sorting:

```bash
find ~/.config -type f -mmin -1440 -printf '%T@ %p\n' 2>/dev/null | sort -nr | head -n 10
```

Here:

- `sort -n` performs a numeric comparison.
- `sort -r` reverses the result.
- Together, `sort -nr` puts the largest/newest numeric timestamps first.

For human-readable output we used:

```bash
find ~/.config -type f -mmin -1440 -printf '%TY-%Tm-%Td %TH:%TM  %p\n' 2>/dev/null | sort -r | head -n 10
```

The date is deliberately formatted as:

```text
YYYY-MM-DD HH:MM
```

This means ordinary text ordering also corresponds to chronological ordering.

---

## 12. Incident INC-001 — Unexpected Configuration Change

### Scenario

Something under `~/.config` may have changed recently.

The initial investigation was:

```bash
find ~/.config -type f -mmin -120 2>/dev/null
```

We could count the matches with:

```bash
find ~/.config -type f -mmin -120 2>/dev/null | wc -l
```

A result of `0` is useful information. A command producing no output does not necessarily mean that the command failed; there may simply be no matching objects.

To display details we experimented with:

```bash
find ~/.config -type f -mmin -120 -exec ls -lht {} \; 2>/dev/null
```

and:

```bash
find ~/.config -type f -mmin -120 -exec ls -lht {} + 2>/dev/null
```

Finally we produced a readable newest-first investigation:

```bash
find ~/.config -type f -mmin -1440 -printf '%TY-%Tm-%Td %TH:%TM  %p\n' 2>/dev/null | sort -r | head -n 10
```

The complete pipeline can be thought of as:

```text
find
  |
  v
locate and filter
  |
  v
format the results
  |
  v
redirect unwanted errors
  |
  v
sort
  |
  v
select the first ten
```

This was our first proper incident-style Linux investigation.

---

## 13. Troubleshooting Lessons

Several mistakes during the module became useful learning exercises.

### Incorrect command

We initially typed:

```bash
md test
```

instead of:

```bash
mkdir test
```

A `command not found` message is useful diagnostic information. Check spelling, aliases, `$PATH`, and whether the required program is installed.

### Accidental commas in filenames

We learned that commas do not normally separate shell arguments. Whitespace does.

### echo and redirection

This:

```bash
echo file1.txt "Linux is fun"
```

only prints its arguments.

To write the text into the file we need redirection:

```bash
echo "Linux is fun" > file1.txt
```

### The /dev/null lesson

This:

```bash
2>/dev/null
```

means:

- `2` — stderr.
- `>` — redirect.
- `/dev/null` — discard the redirected data.

### The sort ~ mistake

Running:

```bash
sort ~
```

caused `~` to expand to `/home/paul`.

`sort` therefore attempted to read the home directory as an input file.

This reinforced the difference between data arriving through a pipe and filenames supplied as command-line arguments.

### The -exec terminator mistake

These are alternatives:

```bash
-exec command {} \;
```

and:

```bash
-exec command {} +
```

They should not be used together.

---

## 14. man, less and Terminal Troubleshooting

Our customised dark terminal exposed a problem where some `man` page headings and search highlighting became difficult to read.

Testing showed that ordinary `less` search highlighting worked correctly.

A useful workaround for the `man` formatting problem was:

```bash
MANROFFOPT='-c' GROFF_NO_SGR=1 man find | less
```

This introduced another important troubleshooting principle:

> Identify which component is producing the problem before changing the whole system.

In this case the chain included:

```text
man -> groff -> less -> terminal
```

We later encountered a similar-looking colour problem with Git, but Git was generating its own colours. We therefore configured Git rather than changing the terminal theme.

---

## 15. Git Fundamentals

The Linux Administration Course is stored in a Git repository:

```text
~/GitHub-Repos/linux-administration-course
```

The basic Git workflow is:

```text
Working directory
       |
       | git add
       v
Staging area
       |
       | git commit
       v
Repository history
       |
       | git push
       v
GitHub
```

Important commands:

```bash
git status
git diff
git add
git diff --staged
git commit
git push
```

`git status` should be used frequently to understand the current state of the repository.

`git diff` shows unstaged changes.

`git diff --staged` shows the changes currently prepared for the next commit.

A good working habit is:

```text
make changes
     |
     v
git status
     |
     v
git diff
     |
     v
git add
     |
     v
git diff --staged
     |
     v
git commit
     |
     v
git push
```

---

## Module 1 Key Principle

Linux administration is not primarily about memorising commands.

The important skill is learning how to break a problem into smaller operations:

```text
locate
  |
  v
filter
  |
  v
format
  |
  v
redirect
  |
  v
pipe
  |
  v
sort
  |
  v
select
```

Small Linux utilities can then be combined to solve much larger administration and troubleshooting problems.

## Module 1 Status

**Completed — Command Line Foundations**



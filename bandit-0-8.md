# OverTheWire — Bandit (Levels 0–8)

**Platform:** OverTheWire (`overthewire.org/wargames/bandit`)  
**Completed:** April 2025  
**Purpose:** Linux command line fundamentals — file navigation, permissions, search tools  

---

## What is Bandit?

Bandit is a wargame designed for complete beginners to Linux. Each level gives you SSH credentials and a challenge — you have to find the password for the next level using Linux commands. There are no hints beyond the man pages.

It's fully free, browser-independent, and runs entirely in your terminal via SSH.

---

## Level 0 — SSH Login

**Goal:** Connect to the game server via SSH.

**Command:**
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# password: bandit0
```

**What I learned:**
- SSH syntax: `ssh user@host -p port`
- `-p` flag specifies a non-default port (default SSH is 22, Bandit uses 2220)
- SSH is the primary way to access remote Linux systems — essential for SOC work

---

## Level 0 → 1 — Read a File

**Goal:** Find the password stored in a file called `readme` in the home directory.

**Command:**
```bash
cat readme
```

**What I learned:**
- `cat` prints file contents to terminal
- Linux home directory (`~`) is where you land after SSH login
- File names are case-sensitive on Linux

---

## Level 1 → 2 — File With a Dash in the Name

**Goal:** Read a file named `-` (a single dash).

**Command:**
```bash
cat ./-
```

**What I learned:**
- A bare `-` is interpreted by most commands as "read from stdin" (keyboard input), not a file
- Prefixing with `./` tells the shell to treat it as a path relative to current directory
- This is a real gotcha — tools like `nc`, `cat`, and `grep` all behave this way

---

## Level 2 → 3 — Spaces in Filename

**Goal:** Read a file named `spaces in this filename`.

**Command:**
```bash
cat "spaces in this filename"
```

**What I learned:**
- Spaces in filenames break commands because the shell splits arguments on spaces
- Fix: wrap the filename in quotes, or escape each space with `\`
- Alternative: `cat spaces\ in\ this\ filename`
- Relevant for log analysis — Windows file paths with spaces are common

---

## Level 3 → 4 — Hidden File

**Goal:** Find and read a hidden file inside a directory called `inhere`.

**Command:**
```bash
cat inhere/.hidden
```

**What I learned:**
- Files starting with `.` are hidden from `ls` by default
- `ls -a` reveals hidden files
- Hidden files are commonly used for config files (`.bashrc`, `.ssh/config`) and sometimes by malware to hide payloads

---

## Level 4 → 5 — Find the Only Human-Readable File

**Goal:** One file in `inhere/` is human-readable (ASCII text). Find it among several binary files.

**Command:**
```bash
file ./-file*
# output shows -file07 is ASCII text
cat ./-file07
```

**What I learned:**
- `file` identifies what type of data a file contains regardless of its name or extension
- Wildcards (`*`) match multiple files at once
- In forensics and malware analysis, `file` is one of the first commands you run on an unknown file
- Never trust file extensions — a `.txt` can contain binary data and vice versa

---

## Level 5 → 6 — Find by Properties

**Goal:** Find a file that is exactly 1033 bytes and not executable, somewhere inside `inhere/`.

**Command:**
```bash
find . -size 1033c ! -executable
```

**What I learned:**
- `find` is one of the most powerful Linux commands for investigations
- `-size 1033c` — `c` means bytes (not kilobytes)
- `!` negates a condition — `! -executable` means "not executable"
- In incident response, `find` is used to locate recently modified files, files with suspicious permissions, or files of unexpected sizes

---

## Level 6 → 7 — Find by Owner and Group

**Goal:** Find a file anywhere on the system owned by user `bandit7`, group `bandit6`, and exactly 33 bytes.

**Command:**
```bash
find / -user bandit7 -group bandit6 -size 33c
```

**What I learned:**
- Searching from `/` searches the entire filesystem
- `-user` and `-group` filter by file ownership — critical for privilege escalation analysis
- The output floods with "Permission denied" errors — add `2>/dev/null` to suppress stderr:
  ```bash
  find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
  ```
- `2>/dev/null` redirects error output to nothing — a technique used constantly in real terminal work

---

## Level 7 → 8 — Search Inside a File

**Goal:** Find the password next to the word "millionth" in a large file called `data.txt`.

**Command:**
```bash
grep "millionth" data.txt
```

**What I learned:**
- `grep` searches file contents for a pattern and returns matching lines
- Essential for log analysis — SOC analysts use `grep` (or its Splunk equivalent) constantly
- `data.txt` in this level has thousands of lines — manual reading is impossible, `grep` finds it instantly
- Useful flags to know: `-i` (case-insensitive), `-n` (show line number), `-r` (recursive search through directories)

---

## Commands Reference

| Command | What it does |
|---------|-------------|
| `ssh user@host -p port` | Connect to remote system |
| `cat filename` | Print file contents |
| `cat ./-filename` | Read file with dash in name |
| `cat "file with spaces"` | Read file with spaces in name |
| `ls -a` | List all files including hidden |
| `file ./*` | Identify file types |
| `find . -size 1033c ! -executable` | Find by size, exclude executables |
| `find / -user X -group Y -size Zc 2>/dev/null` | Find by owner/group, suppress errors |
| `grep "pattern" file` | Search file contents for pattern |

---

## Reflection

Bandit levels 0–8 cover the Linux skills that come up constantly in real SOC work — navigating filesystems, finding files by properties, searching log contents with grep, and understanding file ownership. The `find` and `grep` commands alone are used daily in incident response and log analysis.

**Next:** Continuing Bandit levels 9+ (sorting, unique lines, encoding, compression, remote services).

# 01 — Shell Prompt and Command Syntax

**Official objective:** Access a shell prompt and issue commands with correct syntax
**Phase:** 1 — Essential tools
**Completed:** 2026-10-03 · log: [day-02](../logs/day-02.md)
**Book:** Ch. 02 pp. 41–48

## Concept
The shell (bash on RHEL) is the program that reads a command, runs it and prints the result. The prompt is just the text it shows when it's ready for the next command. The prompt tells me who I am, which machine I'm on, which folder I'm in, and whether I'm a normal user (`$`) or root (`#`). Every command follows the same shape: command, then options, then arguments, separated by spaces.

**Analogy:** a command is a sentence — the command is the verb, options are adverbs (how), arguments are nouns (what). `ls -l /etc` = "list, in long form, /etc".

```
[admin@servera ~]$        [root@serverb tmp]#
 user   host   folder $    root   host  folder #
```

## Key commands

| Command | Meaning |
|---|---|
| `pwd` | print the current folder (always an absolute path) |
| `ls` | list a folder |
| `ls -l` | long listing: type+permissions, links, owner, group, size, date, name |
| `ls -a` | include hidden files (start with `.`) |
| `ls -ld DIR` | show the directory itself, not what's inside |
| `ls -S` / `-t` / `-r` | sort by size / by time (newest first) / reverse |
| `ls -ltr` | newest at the bottom — good for "what changed last?" |
| `cd /abs/path` | absolute path: starts with `/`, works from anywhere |
| `cd rel/path`, `cd ..` | relative path: starts from where I am; `..` = parent |
| `cd` or `cd ~` | home folder |
| `cd -` | previous folder |
| `which CMD` | where a command's file lives |

## How it works on the system
- My shell is `/bin/bash` (`echo $SHELL`). The prompt format lives in the `PS1` variable (`echo $PS1`); RHEL default is `[\u@\h \W]\$`.
- Commands are files, mostly in `/usr/bin` (`which ls` → `/usr/bin/ls`).
- `~` = `/home/<user>`. New home folders get their hidden files from `/etc/skel`.
- `/root` is root's home, `dr-xr-x---`: a normal user falls under "others" (`---`) so gets Permission denied.

## Exam traps
- ⚠️ Case-sensitive: `LS` fails. Spaces matter: `ls-l` and `ls - l` fail.
- ⚠️ Check `$` vs `#` before running anything.
- ⚠️ Read the whole error — often the last line tells you where to look (`Try 'ls --help'`).
- ⚠️ A `>` prompt means the shell is waiting (unclosed quote) — Ctrl+C to escape.
- Survives reboot? Nothing to persist here — but verify every result (`pwd`, re-list).

## Verify it worked
```bash
whoami; hostname; pwd
ls -ld /path/I/expect
```

## Find it offline (no internet on the exam)
- `man ls` → then type `/word` inside it, `n`/`N` next/previous, `q` quit (NOT `man ls /word`)
- `ls --help | less` or `ls --help | grep word`
- `man -k keyword` — find a command by keyword
- `whatis ls` — one-line description
- `/usr/share/doc/`

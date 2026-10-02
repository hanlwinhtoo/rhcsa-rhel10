# Day 02 — Objective 1: Shell Prompt and Command Syntax

**Date:** 2026-10-02 → 2026-10-03
**Objective(s):** Access a shell prompt and issue commands with correct syntax
**Book:** Ghori, *RHCSA RHEL 10* (4th ed.), Ch. 02 — pp. 41–48 (prompt, syntax, ls, pwd, cd), pp. 50–55 (help), p. 51 Table 2-4 (man/less keys)
**Notes:** [01-shell-syntax](../notes/01-shell-syntax.md)

## Why it matters
Every exam task is typed at a shell prompt. One wrong space or capital letter breaks a command and wastes exam time.

## Concept
- **Shell** = the program (bash) that reads what I type and runs it. **Prompt** = the text it prints when it's my turn.
- Analogy: the shell is a waiter; the prompt is the waiter asking "what would you like?" The waiter stays the same, the question repeats.
- `[admin@servera ~]$` → user @ host, current folder, `$` normal user / `#` root.
- Commands read like a sentence: `command [options] [arguments]` = verb, adverbs, nouns.

## Commands

| Command | What it does |
|---|---|
| `whoami` / `hostname` | which user / which machine |
| `echo $SHELL` | show my shell (`/bin/bash`) |
| `cat /etc/redhat-release` | show OS version (RHEL 10.2 Coughlan) |
| `pwd` | print current folder |
| `ls -l` / `-a` / `-ld DIR` | long listing / include hidden / the folder itself |
| `ls -S` / `-t` / `-r` | sort by size / by time / reverse |
| `ls -ltr` | newest file at the bottom |
| `cd DIR`, `cd ..`, `cd` or `cd ~`, `cd -` | go to / up one / home / previous folder |
| `which ls` | where a command lives (`/usr/bin/ls`) |
| `COMMAND \| less` | page long output (console has no scroll-back) |
| `COMMAND \| grep WORD` | show only matching lines (preview, Week 2) |
| Ctrl+C | cancel the current line |

## Lab — what I did
```bash
whoami                     # admin
hostname                   # servera.rhcsa.internal
pwd                        # /home/admin
ls -a                      # .  ..  .bash_logout  .bash_profile  .bashrc
ls -ld /var/log/           # drwxr-xr-x. 12 root root 4096 Oct  2 00:31 /var/log/
ls -la /etc/skel/          # same 3 hidden files: skel is the template for new home dirs
ls -lS /etc/               # largest files first
cd ../../usr/bin           # relative path from /home/admin
cd ~                       # back home
ls -ld /root               # dr-xr-x---. 3 root root 147 Oct  2 00:16 /root
ls -ltr /etc | tail -3     # newest last -> resolv.conf (rewritten when networking starts)
```

`ls /usr/bin` looked like it had no `ls` — it had scrolled off the top. Proved it with `ls -l /usr/bin/ls` and learned to use `| less`.

## Break it and fix it

| Typed | Error | Real cause | Fix |
|---|---|---|---|
| `LS -l` | command not found | case-sensitive: `LS` ≠ `ls` | `ls -l` |
| `ls-l /etc/hostname` | command not found | no space → shell reads `ls-l` as one command name | `ls -l /etc/hostname` (keep the argument) |
| `ls -z` | invalid option -- 'z' | `-z` doesn't exist; 2nd line says `Try 'ls --help'` | check `ls --help` / `man ls` |
| `cd /ect` | No such file or directory | typo in path | `cd /etc` |
| `ls /root` | Permission denied | `/root` is `dr-xr-x---` owned by root; admin = "others" = `---` | needs root (later) |
| `echo "hello` | `>` prompt | not an error: unclosed quote, shell waits for the rest | type `"` + Enter, or Ctrl+C |
| `man ls /reverse` | No manual entry for /reverse | shell passed `/reverse` as a 2nd manual name | `man ls` first, THEN type `/reverse` inside it |

## Exam-style tasks
1. Long-list `/var/log` itself, then everything in `/etc/skel` incl. hidden — `ls -ld /var/log`, `ls -la /etc/skel` — ✅
2. Find the size-sort option offline and use it — `-S` via `ls --help`, `ls -lS /etc/` — ✅ but didn't paste output as proof; dumping all of `--help` is slow → search with `/size` in `man` or `| less`
3. Relative `cd` to `/usr/bin`, back home with one character — `cd ../../usr/bin`, `cd ~` — partial ❌ forgot `pwd` to prove location

Search drill (redo of weak point): `-r` reverse and `-t` time found with `man ls` + `/reverse`, `/modification`; `cp -r/-R` via `cp --help | grep recursive` — ✅

## Exam traps
- ⚠️ Linux is case-sensitive; spaces separate command / options / arguments.
- ⚠️ Check the last prompt character (`$` vs `#`) before typing.
- ⚠️ Always verify: `pwd`, re-list, paste output. Unverified work = lost marks.
- The `.` after permissions (`---.`) is an SELinux label marker, not a permission.

## Offline help
- `man ls` → `/word`, `n`, `N`, `g`, `q` (Table 2-4, p. 51)
- `ls --help | less`, `ls --help | grep word`
- `man -k keyword` finds commands, not options (run `sudo mandb` if it says "nothing appropriate")
- `whatis pwd`, `/usr/share/doc/`

## Mastered / still weak
- Mastered: reading the prompt, command syntax, `ls` options, `pwd`, `cd` absolute/relative, searching inside `man`/`less`
- Weak: **verification** (proving the result) and **explaining the real cause** of an error instead of repeating it

## Next session
Objective 11 — Locate, read, and use system documentation including man, info, and files in /usr/share/doc. Read Ch. 02 pp. 50–55 first.

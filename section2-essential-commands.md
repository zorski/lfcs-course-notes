## SECTION 2: Essential Commands

### 4. Log into local & remote graphical and text mode consoles
* consoles
* virtual terminal - ctrl+alt+f2 activates virtual terminal (vty2)
* terminal emulator

### 7. Read and use system documentation
Ways to access help, manuals and documentation in Linux

1. `--help` parameter with condensed help, e.g.:
* `ls --help`
* `journalctl --help`

2. `man` command
* `man <command>`, e.g. `man journalctl`
* documentation in Linux is divided into **sections** (`man <section number> <command>`)
    - 1 Executable programs or shell commands
    - 2 System calls (functions provided by the kernel)
    - 3 Library calls (functions within program libraries)
    - 4 Special files (usually found in /dev)
    - 5 File formats and conventions, e.g. /etc/passwd
    - 6 Games
    - 7 Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7), man-pages(7)
    - 8 System administration commands (usually only for root)
    - 9 Kernel routines [Non standard]
* `man -K` - global apropos, brute-force search through whole man page text

3. `apropos` command searches through man pages. It looks through short descriptions of commands
* `apropos director`
* `sudo mandb` to initalize databae (index)
* `apropos -s 1,8 director` - only search in sections 1 and 8 (look above for listed sections)

4. tab completion




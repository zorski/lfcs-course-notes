## SECTION 2: Essential Commands

### 4. Log into local & remote graphical and text mode consoles
* consoles
* virtual terminal - ctrl+alt+f2 activates virtual terminal (vty2)
* terminal emulator

### 7. Read and use system documentation
Ways to access help, manuals and documentation in Linux

#### The `--help` parameter 
This parameter displys condensed help for a given command, e.g.:
`ls --help` or `journalctl --help`

#### The `man` command
This command display a detailed `man <command>`, e.g. `man journalctl`. Documentation in Linux is divided into **sections** (`man <section number> <command>`):
* **1** - Executable programs or shell commands
* **2** - System calls (functions provided by the kernel)
* **3** - Library calls (functions within program libraries)
* **4** - Special files (usually found in /dev)
* **5** - File formats and conventions, e.g. /etc/passwd
* **6** - Games
* **7** - Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7), man-pages(7)
* **8** - System administration commands (usually only for root)
* **9** - Kernel routines [Non standard]

The `man -K` command and parameter performs a global search in a whole content of man page (it can be time consuming).

3. `apropos` command searches in short descriptions of man pages (same as `man -k`), e.g.: `apropos director` or `apropos -s 1,8 director` for only searching in specific sections (1 and 8 in this case)

4. tab completion

### 11. Create, delete, copy and move files and directories

* `ls -a` - list all (`-a`) files (including hidden files)
* `-l` - long listing format
* `-h` - human readable format (has to be combined with `-l`)
* `pwd` - print current directory
* `cd` - change directory
* `cd -` - previous directory
* `cd /` - root (top)
* `cd ..` - one up
* `touch` - creates file (and change file timestamps)
* `cp [source] [destination]` - copy file
* `cp -r [source] [destination]` - copy recu* rsively
* `mv [source] [destination]` - move files


Linux files are organized in tree structure 

**TODO**: practice cp / mv a bit





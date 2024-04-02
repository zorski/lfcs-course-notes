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

### 12. Create and manage hard links
file that points to the same **inode**

`stat` - file info including inodes
`ln [path_to_target] [path_to_link_file]` - creates hard link

#### limitations for hard links:
* only to files (not directories)
* only on the same filesystem

### 13. Create and manage soft links
file that points to another file or directory on the same or different filesystem

`readlink` - shows where soft link is pointing

### 14. Lab: Files,directories, hard and soft links
`ls --full-time`
`mkdir -p`

### 15. List, set, and change standard file permissions
`chgrp [group] [filename]` - change group to which the file belongs
`chown` - change owner to which the file belongs (e.g.`sudo chown user file.jpg`)

#### permissions
* for files:
    - `r` / read file
    - `w` / write to file
    - `x` / execute file
    - `-` / no permissions
* for directories:
    - `r` / read directory (`ls Pictures`)
    - `w` / write to directory (`mkdir Picture/Family`)
    - `x` / execute into (`cd Pictures`)
    - `-` / no permissions

Permissions are evaluated in linear fashion from left to right.
To change permissions `chmod` command is used. Permissions can be specified in few ways, e.g.:
* `chmod u+rw dog.jpg` - user permissions added (`rw`) for file dog.jpg
* `chmod g-r dog.jpg` - group permissions removed (`r`) from file dog.jpg
* `chmod o+rwx dog.jpg` - others permissions added to file dog.jpg
* `chmod u=rw dog.jpg` - users permissions set to exactly `rw`
* `chmod g= dog.jpg` - clear permissions from group (same as `chmod g-rwx dog.jpg`)
* `chmod 640 dog.jpg` - sets `u=rw,g=r,o=` on file dog.jpg

`stat dog.jpg` - displays permissions

* `r = 4, w = 2, x = 1`

### 16. SUID, SGID and sticky bit
When SUID is set, the file will be executed as the user ID of the owner of the file instead of user ID which actually running it.

#### Setting SUID permission:
`chmod 4664 suidfile` - `4` at the beggining is setting the SUID bit. When `ls -lh` is executed file with SUID set looks like that:
`-rwSrw-r-- 1 mz mz    0 Apr  2 20:55 testsuidfile`
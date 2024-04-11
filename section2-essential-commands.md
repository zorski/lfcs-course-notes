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
When SUID is set, the file will be executed as the user ID of the owner of the file instead of user ID which actually running it. SGID is the same concept but applies to group permissions. Sticky bit allows only the user owner to remove file (no matter the permissions on the directory)

#### Setting SUID/SGID permissions:
* `chmod 4664 suidfile` - `4` at the beggining is setting the SUID bit. When `ls -lh` is executed file with SUID set looks like that:
* `-rwSrw-r-- 1 mz mz    0 Apr  2 20:55 testsuidfile` - when there is not execute permission (capital `S`)
* `-rwsrw-r-- 1 mz mz    0 Apr  2 20:55 testsuidfile` - when `x` was set (lowercase `s`)
* `chmod 2664 sgidfile` - `2` sets SGID
* `chmod 1664 stickydirectory` - sets sticky bit

#### finding SUID/SGID files:
`find . -perm /4000` or `find . -perm /2000` (group) or `find . -perm /6000` (both)

### 17. Search for files

* `find /usr/share -name *.jpg` - all `jpg` files in `/usr/share/`
* `find -name file1.txt` - find `file.txt` in current directory
* `find -iname felix` - case insensitive
* `find -mmin 5` - find files modified exactly five minutes ago
* `find -mmin -5` - last 5 minutes
* `find -mmin +5` - before five minutes ago (and to infinity :) )
    - read vs change vs modified here: https://stackoverflow.com/a/3385248
    - modified - create or edit
    - change - metadata change
* `find -size +512k` - 512 kb or more
* `find -name "f*" -o -size 512k` - name starts with "f" OR size is equal to 512 kilobytes
    - without `-o` it is a AND operator by default
* `find -not -name "f*"` - files not starting with "f" (NOT opeator, could be also `\!`)
* `find -perm 664` - files with exactly 664 (u=rw,g=rw,o=r) permissions
* `find -perm -644` - at least 664
* `find -perm /664` - any of these permissions (e.g. u=r will match)

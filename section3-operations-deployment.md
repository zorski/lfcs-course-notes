## SECTION 3: OPERATIONS DEPLOYMENT

### 34. Boot, reboot and shutdown a system safely

* `systemctl reboot`
* `systemctl poweroff`
* `systemctl reboot --force`
* `systemctl poweroff --force`
* `systemctl poweroff --force --force`
* `shutdown 02:00` - shutdown at 2AM (24 hour format)
* `shutdown +15` - shutdown after 15 minutes
* `shutdown -r +15` - reboot after 15 minutes
* `shutdown -r +1 'Scheduled restart to do an offline-backup of our database'` - reboot after 1 minutes with *wall* message

### 35. Boot or change system into different operating modes

* `systemctl get-default` - default target
* `systemctl set-default multi-user.target` - change default target
* `systemctl isolate graphical.target` - change to target (without changing default one and rebooting)
* `systemctl isolate emergency.target` - change to emergency.target (few programs as possible, root filesystem is read-only) which could be useful for debugging (password must be set to **root** user)
* `systemctl isolate rescue.target` - rescue.target loads a bit more programs than emergency.target 

### 36. Install, configure and troubleshoot bootloaders

#### bios
grub2-mkconfig
grub2-install /dev/sda

#### EFI
dnf reinstall grub2-efi grub2-efi-modules shim

#### changes to grub config
edit `/etc/default/grub`  and  run `sudo grub2-mkconfig -o /boot/grub2/grub.cfg` (for bios based) or `grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg` (for EFI)

### 38. Use scripting to automate system maintenance tasks
* `#!/bin/bash` - shebang
* `#!/usr/bin/env bash` - another shebang
* `chmod u+x script.sh` - grant execute permission
* `help` - list builtins
* `cat /etc/cron.hourly/0anacron` - cheatsheet for common syntax

### 39. Manage the startup process and services (In Services Configuration)
* `systemctl edit --full sshd.service` - edit service config
* `systemctl revert sshd.service` - revert to defaults
* `systemctl status sshd.service` - status of the service (loaded,enabled,active,logs)
    - enabled
    - active
    - PID

* `systemctl stop sshd.service`, `systemctl start sshd.service`, `systemctl restart sshd.service`
* `systemctl reload sshd.service`
* `systemctl reload-or-restart sshd.service`
* `systemctl disable sshd.service`, `systemctl enable sshd.service` - enable to run automtically at boot
* `systemctl enable --now sshd.service` - enable and start
* `systemctl mask atd.service` - service cannot be enabled or started
* `systemctl list-units --type service --all` - list all services

### 40. Create systemd Services

* `systemd-cat` - receives a message and logs it
    - `echo "MyApp Started" | systemd-cat -t MyApp -p info`
    - `echo "MyApp Crashed" | systemd-cat -t MyApp -p err`

#### unit files
* `man systemd.service` - help
* `ls /lib/systemd/system` - possbile sample unit files
* `/etc/systemd/system` - unit files are located here
* `systemctl daemon-reload` - reload systemd

### 42. Diagnose and manage processes
`nice -n 11 bash` - assign a value of 11 to bash
`ps` - inspect processes, supports two syntaxes (with and without dash: unix and bsd syntax)
`ps l` - displays nice values for processes
`ps lax` - all processes and extended information (incl. nice values)
`ps fax` - tree like view of processes with parent/child relationship\

    - `ps aux` - all processes (in BSD syntax)
    - processes wrapped in brackets are kernel processes running inside a of a privileged area of Linux Kernel
    - `ps -U username` - processes by user
`top` - continously watch processes

#### nice

Regular user can only assign nice values from 0 to 19. Root user can assign negative values. The **nice value** determines the priority of the process. The higher the value, the lower the priority--the "nicer" the process is to other processes. The default nice value is 0 on Linux workstations. Relation between nice value and priority is `PR = 20 + NI`. In Linux system priorities are **0** to **139** in which 0 to 99 for real time and 100 to 139 for users. The value of `PR = 20 + (-20 to +19)` is 0 to 39 that maps 100 to 139. In short nice value can *adjust* the PR value within a reasonable range.

1. To launch a program and set a nice value:
`nice -n nice_value program_name`

2. To change a nice value of an already running process:
`renice -n nice_value -p process_id`

Regular user can only lower nice value once. 

#### signals
List of signals can be displayed using `kill -L`, signals can be used with `kill` command, e.g.:
* `kill -SIGHUP 1147`
* `kill -SIGKILL 1147`
* `pkill process_name`

#### backgrounding and foregrounding
* `CTRL + Z` - pauses a progress (programm is frozen in background)
* `fg` - return to app (bring to foreground)
* `bg` - put stopped process to background
* `sleep 300 &` - backgrounds a process and it still runs
* `jobs` - list backgrounded processes

#### audit 
* `lsof -p PID` - currently opened files and directories by specified process (list open files)

* `lsof /var/log/messages` - check which processes are currently using directory or file


### 43. Locate and analyze system log files
* `tail -F` - follow mode
* `journalctl` - to display logs
* `journalctl /usr/bin/sudo` - all generated by `sudo`
* `journalctl -u sshd.service` - all generated by service
* `journalctl -f` - all logs generated with follow mode (live)
* `journalctl -p info -g '^b'` - priority `info` and starts with `b`
* `journalctl -S 02:00` - since 2 am 
* `journalctl -S 01:00 -U 02:00` - between 1 and 2 am
* `journalctl -b 0` - current boot logs (previous boots: `-1` and so on)
    - `mkdir /var/log/journal` to persist logs from previous boots

#### logs priority
* emerg
* alert
* crit
* err
* warning
* notice
* info
* debug

* `last` - list logins on the system
* `lastlog` - lists when user logged last time


### 45. Schedule tasks to run at the set date and time
* cron
* anacron
* at

#### cron
`cat /etc/crontab` - example format to lookup

`*` - match all possible values
`,` - multiple values
`-` - range
`/` - step


```
35 6 * * * /usr/bin/touch test_passed // everyday at 06:35
0 3 * * 7 /usr/bin/touch test_passed // every sunday at 3:00
0 * * * * /usr/bin/touch test_passed // once every hour
0 3 15 * * /usr/bin/touch test_passed // 15th day of the month at 3:00
```
##### commands
`crontab -e` - edit current user's cron table
`crontab -l` - list cron jobs
`sudo crontab -e -u jane` - edit crontab of user `jane` (only as `root`)
`crontab -r` - remove

Cronjobs can also be run by placing them in following directories:
* `/etc/cron.daily/`
* `/etc/cron.hourly/`
* `/etc/cron.monthly/`
* `/etc/cron.weekly/`

Whatever is placed in these directories will run accordingly.

#### anacron
anacron will run after schedule is missed (e.g. server was powered off).
`sudo vim /etc/anacrontab`
```
7   10  test-job    /usr/bin/touch job_run
```

#### at
`at` can be used to run command at specific time and/or date. Command is run with `at` and date or time, command is specified and accepted using `CTRL+d`.

```
at 15:00
at> /usr/bin/touch at_job

at 'August 20 2024'
at> /usr/bin/touch august_job


at 'now + 30 minutes'
at 'now + 3 days'

```

`atq` commands lists jobs which are still in queue (at queue)  
`at -c <id>` - prints **c**ontents of at jobs  
`atrm <id>` - remove at job  




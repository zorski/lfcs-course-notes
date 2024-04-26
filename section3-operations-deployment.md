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
`#!/bin/bash` - shebang
`#!/usr/bin/env bash` - another shebang
`chmod u+x script.sh` - grant execute permission
`help` - list builtins
`cat /etc/cron.hourly/0anacron` - cheatsheet for common syntax

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
`man systemd.service` - help
`ls /lib/systemd/system` - possbile sample unit files
`/etc/systemd/system` - unit files are located here

`systemctl daemon-reload` - reload systemd
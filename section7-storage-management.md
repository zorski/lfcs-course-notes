### 1. List, create, delete, and modify physical storage partitions

* `lsblk`
* `gpt` or `mbr` (dos)
* `fdisk --list`

### 2. Configure and manage swap space
* `sudo swapon --show`
* `mkswap /dev/vdb3`

#### use file as swap
1. `sudo dd if=/dev/zero of=/swap bs=1M count=128` - 128M of swap file
2. `sudo chmod 600 /swap`
3. `sudo mkswap /swap`
4. `swapon --show` will show file as TYPE

### 3. Configure systems to mount file systems on demand
`autofs`

#### autofs
* `sudo dnf install autofs`
* `systemctl start autofs.service` and enable


#### nfs 
1. install: `sudo dnf install nfs-utils`
2. start and enable (systemd)
3. edit: `sudo vim /etc/exports`:
```
/etc 127.0.0.1(ro)
```
4. `systemctl reload nfs-server.service`

#### autofs config
1. edit `/etc/auto.master`
```
/shares /etc/auto.shares --timeout=400
```
* `/shares` directory where share will be mounted
* `/etc/auto.shares` auto-mouting options for this share are defined here

2. edit `/etc/auto.shares`
```
mynetworkshare -fstype=auto 127.0.0.1:/etc
```
3. Share will be automounted under `/shares/mynetworkshare`
* filesystem type will be automatically determined

4. `systemctl reload autofs`

### 4. Evaluate and compare the basic file system features and options
1. `findmt` - list everything currently mounted on the system
    * `findmt -t xfs,ext4`
    * it show mount options ('rw', 'noexec', 'nosuid')

2. `sudo mount -o ro,noexec,nosuid /dev/vdb2 /mnt`
3. remoount with different options with: `sudo mount -o remount,rw,noexec,nosuid /dev/vdb2 /mnt`

#### filesystem specific mount options
`man xfs` - check MOUNT OPTIONS section, eg. `allocsize`

#### mount options at boot time
`/etc/fstab`:
```
/dev/vdb1 /mybackups xfs ro,noexec 0 2
```

### 5. Use remote filesystems - NFS
* `sudo apt install nfs-kernel-server`
* `sudo vim /etc/exports`
```
/srv/homes hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
```

* `rw`/`ro` - readwrite and read-only
* `sync` - synchronous writes (vs asynchronous writes)
* `no_subtree_check` - disables subtree checking
* `no_root_squash`

* `sudo exportfs -r` - refresh exports 
* `sudo exportfs -v` - list current exports

#### tips
* wildcards can be used:
```
/etc *.example.com(ro,sync,no_subtree_check)
/var/log *(ro,sync,no_root_squash)
```

#### client utilities
* `sudo apt install nfs-common`
* `sudo mount nfsserver01:/etc /mnt` - mount nfs share
* `sudo umount /mnt`

### 6. Use Network Block Devices
This allow sharing block devices through network

#### server
1. `sudo apt install nbd-server` - install Network Block Devices package
2. `sudo vim /etc/nbd-server/config`:
```
[generic]
    includedir = /etc/nbd-server/conf.d
    allowlist = true
[partition1]
    exportname = /dev/vda1
```

3. Restart nbd deamon `sudo systemctl restart nbd-server.service`
4. Help `man nbd-server`

#### client
1. `sudo apt install nbd-client`
2. `sudo modprobe nbd` load module into kernel
    - `sudo /etc/modules-load.d/modules.conf` to load it permanently (add `nbd` line to the file)
3. `sudo nbd-client 192.168.30.4 -N partition1` - to attach network block device
4. `sudo mount /dev/nbd0 /mnt`
5. to detach:
    - `sudo umount /mnt`
    - `sudo nbd-client -d /dev/nbd0`
6. to list available exports `sudo nbd-client 192.168.30.4 -l` 
    - `allowlist = true` option made listing possible 

### 7. Monitor storage performance
* `sysstat` package:
    - `iostat` and `pidstat`

* `iostat`:
    - `tps` - transfers per second (read or write something)


### 8. Manage and configure LVM storage
* PV - physical volume
* VG - volume group
* LV - logical volume (similar to partition)

#### add disk as PV to LVM
1. `sudo pvcreate /dev/sdc /dev/sdd` - adds `/dev/sdc` and `/dev/sdd`
2. `sudo pvs` - list currently added physical volumes
    - `PFree` - show unused storage space

#### create and add Volume Group (vg)
1. `sudo vgcreate my_volume /dev/sdc /dev/sdd` - create and add disks to volume group
    - it will be seen as a single continous disk (e.g. 5G + 5G = 10G)
2. to expand:
    - `sudo pvcreate /dev/sde` - add new disk as a physical volume
    - `sudo vgextend  my_volume /dev/sde` - extends existing volume group with additional disk (pv)
3. to remove:
    - `sudo vgreduce my_volume /dev/sde` - removing PV from VG
    - `sudo pvremove /dev/sde` - remove disk from LVM entirely

#### create logical volume
1. `sudo lvcreate --size 2G --name partition1 my_volume`
    - *partition1* (lv) of size 2G was created as a part of *my_volume* (vg)
2. `sudo lvcreate --size 6G --name partition my_volume`
3. `sudo lvs` - list logical volumes
4. to resize: 
    - grow: `sudo lvresize --extents 100%VG my_volume/partition1` (extends to 100% of VG)
    - shrink: `sudo lvresize --size 2G my_volume/partition1`


* `sudo lvdisplay` - details on logical volumes (e.g. get LV path / name)

* `sudo mkfs.xfs /dev/my_volume/partition1` to create filesystem on LV

#### resizing on already formatted partition
`sudo lvresize --resizefs --size 3G my_volume/partition1` - resizing LV and filesystem as well

refs: `man lvm`:
* SEE ALSO section for all commands

### 9. Create and configure encrypted storage

1. Encrypt disk in plain mode: `sudo cryptsetup --verify-passphrase open --type plain /dev/vde mysecuredisk`
    - `--verify-passphrase` - ask for passphrase twice
    - `open` - action to perform, here `open` device for reading encryped data and writing to it.
    - `type plain` - customizes `open` action to use `plain` encryption scheme
    - `mysecuredisk` - name of the disk (can be chosed freely)

2. Disk will be available under `/dev/mapper/mysecuredisk` and can be used as regular disk device
    - write: `/dev/mapper/mysecuredisk` -> *encryption* -> `/dev/vde`
    - read: `/dev/vde` -> *decryption* -> `/dev/mapper/mysecuredisk`

3. `close` device: `sudo cryptsetup close mysecuredisk`

#### LUKS encryption
1. `sudo cryptsetup luksFormat /dev/vde` - to setup
2. `sudo cryptsetup luksChangeKey /dev/vde` - changes passphrase
3. `sudo cryptsetup open /dev/vde mysecuredisk` - to open device with LUKS
4. `sudo cryptsetup close mysecuredisk`

#### to encrypt part of the disk
1. `sudo cryptsetup luksFormat /dev/vde2` - to setup just specify partition instead of block device

13/17: "Setup `/dev/vde` as an encrypted disk. The mapped device should be called `secretdisk`.
Use plain type encryption (not LUKS) and the password must be `S3curepass`."
* `sudo cryptsetup --type plain --verify-passphrase secretdisk /dev/vde` 


### 10. Create and manage RAID devices

#### RAID Levels
* level 0 (stripped) - total space is a sum of all disks
* level 1 (mirror)
* level 5 (parity) - all disks have parity data on all of them
    - one disk can be lost
* level 6 () - 4 disks required
* level 10 (stripped + mirror)


#### creating RAID arrays
`mdadm` tools is used to create RAID arrays

```
sudo mdam --create /dev/md0 --level=0 --raid-devices=3 /dev/vdc /dev/vdd /dev/vde
sudo mkfs.ext4 /dev/md0
sudo mdam --stop /dev/md0
```

When boots up Linux scans up for superblock of all devices, it means it will try to check for information if disks are not a part of RAID array. If such information is found, Linux will try to reassemble it and create randomly numbered  array (e.g. `/dev/md123`).

In order to prevent, data can be wiped on superblocks: `sudo mdadm --zero-superblock /dev/vdc /dev/vdd /dev/vde`.

#### adding spare disks
```
sudo mdam --create /dev/md0 --level=1 --raid-devices=2 /dev/vdc /dev/vdd --spare-devices=1 /dev/vde
```

#### addding or removing disk for already existing RAID array
```
sudo mdadm --manage /dev/md0 --add /dev/vde

sudo mdam --manage /dev/md0 --remove /dev/vde
```

* `cat /proc/mdstat` - RAID info

### 11. Create, manage and diagnose advanced file system permissions
`setfacl` - set advanced file permissions 
1. `sudo setfacl --modify user:aaron:rw examplefile`
2. file which have additional acls set have `+` at the end of permission string
    - ``
3. `getfacl` - to list permissions
4. `sudo setfacl --recursive -m --modify user:aaron:rwx dir/` - recursively apply to directory and everyting below

#### file and directory attributes

1. `append-only` attribute on a file
    - `chattr +a example_file` to add
    - with this attribute data can only be added to file, but **not** modified
    - `chattr -a example_file` to remove
2. `immutable` attribute:
    - `chattr +i example_file`
    - file with this attribute can be changed in any way 
    - not even `root` can change it (incl. removal)
3. Check for attributes with `lsattr example_file`
4. Other attributes `man chattr`

### 12. Setup user and group disk quotas for filesystems
Quota limit 

1. `sudo dnf install quota` - to install quota packages
2. edit `/etc/fstab`
```
/dev/vdb1 /mybackups xfs defaults,usrquota,grpquota 0 2
```
3. reboot server
    - on `xfs` it is already set adn ready to use
    - on other filesystems (like `exqt4`) additional step are required
4. On `ext4`:
    - `sudo quotacheck --create-files --user --group /dev/vdb2` will create two files `aquota.group` and `aquota.user`. In these files system will keep track how much storage space each user/group is using.
    - `sudo quoaton /mnt/` - announces to the system that disk quotas should be enabled on one or more filesystems. The filesystem quota files must be present in the root directory of the specified filesystem and be named either aquota.user (for version 2 user quota), quota.user (for version 1 user quota), aquota.group (for version 2 group quota), or quota.group (for version 1 group quota). 

5. `sudo quota --user aaron` - checks quota for user
6. `sudo edquota --user aaron` - edit quotas for user
7. `sudo edquota --edit-period` - to edit grace periods
8. `sudo edquota --group adm` - to edit quota for group


### MOCK EXAM 1
* suid once again
* sticky bit
* find permission and size searches
* 7th - case insensitive?
* zip  
 -  sudo unzip backup.zip -d /opt/restoredzip/
* limits.conf
* removing user from group
    -  `sudo gpasswd -d jack wheel`
* diff options
* ip command adding routes
* lvm man pages

#### issues:
Q. 3 : subtask "Copied "file4" to the "/opt/newdir" directory?" always fails
Q. 5 : subtask "Found user executable files?" always fails
Q. 6 : this task always fails
Q. 13 : i think that in this task you require spaces and not tabs, so it failed for this "jane    hard    nproc   30" from me
Q. 19 : failed even though correct pid was in the "/opt/pidof22.txt"
Q. 22 : subtask "frozenfile" file is no longer immutable?" failed even though file is not longer immutable (checked with lsattr)

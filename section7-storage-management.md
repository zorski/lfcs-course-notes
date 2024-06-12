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





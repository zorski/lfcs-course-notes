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

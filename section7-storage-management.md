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
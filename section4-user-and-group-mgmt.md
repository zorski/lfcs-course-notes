## SECTION 4: USER AND GROUP MANAGEMENT

### 57. Create, delete and modify local user accounts
* `sudo useradd john`:
 - new user `john` is created
 - new group also called `john` is also created
 - group `john` is set a user's primary group
 - home directory is created 
 - default shell is set
 - all files from `/etc/skel` are copied to user's home directory
 - account will never expire

* `useradd --defaults` - to explore defaults
* `cat /etc/login.defs`

* `sudo passwd john` - to set password for user

* `sudo userdel john` - delete user account, group might be deleted, home directory stays
* `sudo userdel -r john` to remove all

To modify defaults 
* `sudo useradd --shell /bin/zsh --home-dir /home/otherdirectory john`
* `sudo useradd -s /bin/zsh -d /home/otherdir john` - short form

#### /etc/passwd
In `/etc/passwd` information about user is stored
```
wieczork:x:1107:1107:Mariusz Wieczorkiewicz:/home/wieczork:/bin/bash
...
```
* username
* placeholder for password (`x`) which is stored in `/etc/shadow` in salted and encrypted form
* user id
* group id
* user id info (GECOS)
* home directory
* command/shell

#### modify users:
To modify home directory:
```bash
usermod --home /home/other --move-home john
usermod -d /home/other -m john #shorthand

```
To change the username from `john` to `jane`
```bash
sudo usermod --login jane john
sudo usermod -l jane john #shorthand
```

To change user's login shell
```bash
sudo usermod --shell /bin/zsh jane
sudo usermod -s /bin/zsh jane
```

To lock account (disable without deleteing) and (`--unlock` / `-U`):
```bash
sudo usermod --lock jane
sudo usermod -L jane
```

To expire user's account:
```bash
sudo usermod --expiredate 2024-05-29 jane # date format: YEAR-MONTH-DAY
# to expire user's account immediately use past date or 1
usermod -e 1 jane

# to remove expiration flag execute same command but with "" instead of date
sudo usermod --expiredate "" jane
```

Password expiration (forces user to change password):
```bash
sudo chage --lastday 0 jane

sudo chage --lastday -1 jane # unexpire password
sudo chage --maxdays 30 jane # password expires every 30 days
sudo chage --maxdays -1 jane # never expires
sudo chage --list jane # accounts aging information
```

To delete 
```bash
sudo userdel -r jane
sudo groupdel john
```




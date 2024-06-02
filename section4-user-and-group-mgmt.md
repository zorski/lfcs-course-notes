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

### 58. Create, delete, and modify local groups and group memberships

* Primary group

```bash
# create user john, create group developers and add john to developers
sudo useradd john
sudo groupadd developers
sudo gpasswd --add john developers
```

```bash
# remove john from developers group (argument order "user group")
sudo gpasswd --delete john developers

# set developers group as john primary group (argument order "group user")
sudo usermod -g developers john

# rename group
sudo groupmod --new-name programmers developers 

# remove group, but it cannot be done if group is primary
sudo groupdel programmers

# setting back primary group
sudo usermod --gid john john

# delete successful
sudo groupdel programmers 
```

### LAB 
Set the jane user account to expire on March 1, 2030.
`sudo chage --expiredate "March 1, 2030" jane`

Create a system account called apachedev (what is system account??)
`sudo useradd -r apachedev`

Mark the password for jane as expired to force her to immediately change it the next time she logs in.
`sudo chage --lastday 0 jane`

`sudo groupadd -g 9875 cricket`

Create a user sam with UID 5322, also make it a member of soccer group.
`sudo useradd -u 5322 -G soccer sam`

`sudo usermod --gid rugby sam`

Lock the user account called sam -> sudo usermod -L sam`sudo usermod -L sam`


### 59. Manage system-wide environment profiles
`printenv` or `env` lists current environment variables

`.bashrc` - user's variables? (how about `.bash_profile`)
`/etc/environment` - set variable every time user logs in (system wide / globally available)
`/etc/profile.d` - shell scripts to run at startup?

When setting variable in `.bashrc` (and similar):
* `export VAR_NAME="var value"` - makes variable available to all subprocesses of this shell (e.g. scripts)
* without export keyword, variable is only available to shell

### 60. Manage template user environment
`/etc/skel/` - files put in this directory will be used as a template for new users - files will be copied to new user's home directory.

### 61. Configure user resource limits
File for setting up limits is `/etc/security/limits.conf`
```
#<domain>   <type>  <item>  <value>
john        hard    nproc   10
@developers soft    nproc   20
*           soft    cpu     5
```
`<domain>`:
* user
* group (with `@group` syntax)
* wildcard (`*`)
* *the wildcard %, can be also used with %group syntax, for maxlogin limit*

`<type>`:
* hard - cannot be overwritten by regular user. It is top/max limit
* soft - actually enforced by kernel, but can be overwritten by regular user up to the value of the hard limit.

`<item>`, e.g:
* cpu - limit for cpu time in minutes
* nproc - number of processes
* maxlogins

to view limit for current session `ulimit -a`

### 62. Manage user privileges
* `wheel` group
* `/etc/sudoers` -> `visudo`

* "**root** ALL=(ALL:ALL) ALL" - the first field indicates the username that the rule will apply to (`root`).
* root **ALL**=(ALL:ALL) ALL - indicates that this rule applies to **all hosts**.
* "root ALL=(**ALL**:ALL) ALL" - indicates that the root user can run commands as **all users**.
* "root ALL=(ALL:**ALL**) ALL" - indicates that the root user can run commands as **all groups**.
* "root ALL=(ALL:ALL) **ALL**" - indicates these rules apply to **all commands**.


### 63. Manage access to the roo account

`sudo --login` - allows to login as root even if root is locked
`su -` - requires unlocked and password set for root 

`sudo passwd --lock root` - can be used to lock root, however, it is important to make sure users can sudo. It also doesn't prevent already established logins (like via ssh keys)



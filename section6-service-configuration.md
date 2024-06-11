## SERVICE CONFIGURATION

### 1. Configure a caching DNS server
`sudo dnf install bind bind-utils` 
`man named.conf`

#### editing bind config
`/etc/named.conf`
```
listen-on port 53 { 127.0.0.1;};
allow-query { localhost; }; # could be any or other ips
recursion yes;

```
1. start bind `systemctl start named.service` and enable it
2. `sudo firewall-cmd --add-service=dns`

### 2. Maintaining a DNS zone
in `/etc/named.conf`
```
zone "exmaple.com" IN {
    type master;
    file "example.com.zone";
}

```
in `/var/named/example.com.zone`:
```
$TTL 1H
@       IN  SOA  @  administrator.example.com. (
                                        0   ; serial
                                        1D  ; refresh
                                        1H  ; retry
                                        1W  ; expire
                                        3H ); minimum
@               NS      ns1.example.com.
@               NS      ns2.example.com.
ns1             A       10.11.12.9
ns2             A       10.11.12.10
@               A       203.0.113.15
www             CNAME   example.com.
example.com.    MX 10   mail.example.com.
example.com.    MX 20   mail2.example.com.
mail            A       203.0.113.80
mail2           A       203.0.113.81
server1         AAAA    fe80::6a37:61fc:e097:eb0f
example.com.    TXT     "Anything!"

```


### SOA record parameters
* serial - must be increment anytime change is made to a zone. Secondary DNS server can compare it with master servers serial and know when to refresh local copy.
* refresh - how to long to wait to check for refresh (seconds)
* retry - how long to wait after failed refresh attempt
* expire - how long to wait until marking primary dns server as expired and stopping responding to queries for that zone
* minimum - how long other servers should cache negative responses


### 3. Configure email aliases
`/var/spool/mail/aaron`

1. Install postfix (`sudo dnf install postfix`)
    - start and enable (`systemctl`)
2. `sendmail aaron@localhost <<< "Hello, testing."`
3. `cat /var/spool/mail/aaron`
4. Edit aliases `sudo vim /etc aliases`:
```
advertising: aaron
contact: aaron,jane,john
marketing: aaron.aronofsky@gmail.com
```
5. Inform postfix about new aliases `sudo newaliases`
6. `sendmail advertising@localhost <<< "Hello, testing."`

### 4. Configure an IMAP and IMAPS service

1. `sudo dnf install dovecot`
2. enable and start
3. allow through firewall
    - `sudo firewall-cmd --add-service=imap`
    - `


4 task : sudo dnf install s-nail


### 5. Configure SSH servers and clients
* `/etc/ssh/sshd_config` - config for ssh daemon
* `man sshd_config`
* `sudo systemctl reload sshd.service`
* remake keys???
* `ssh-keygen`
* `ssh-keygen -R <hostname/ip>`


### 6. Restrict access to the HTTP proxy server
1. `sudo dnf install squid`
2. `sudo firewall=cmd --add-service=squid`


### 7. Restrict access to the HTTP proxy server
Watch something about SQUID?
`/etc/squid/squid.conf`

### 8. Configure an HTTP server
```
sudo dnf install httpd
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=https
sudo firewall-cmd --runtime-to-permanent
```
* `man httpd.conf`
* `/etc/httpd/conf.d` and 
* `/etc/httpd/conf/httpd.conf` - primary config file
* `apachectl configtest` - check config files for correctness
* `/etc/httpd/conf.d/ssl.conf` - default SSL settings
* `/etc/httpd/conf.modules.d/` - modules (`LoadModule` directives etc.)

### 9. Configure HTTP server log files
* `/etc/httpd/log/error_log` and `/var/log/httpd/error_log`
* loglevels: warn is default

### 10. Restrict access to a web page
* `AllowOverride` 
* `Options Indexes FollowSymLinks`
* `Require all granted` `valid-user`
* `htpasswd`
* `Require ip <ip>`

Sample directive for basic auth on subpage:
```
<Directory "/var/www/html/admin/>
    AuthType Basic
    AuthBasicProvider file
    AuthName "Secret admin page"
    AuthUserFile /etc/httpd/passwords
    Require valid-user
</Directory>
```

### 11. Configure a database server
1. `sudo dnf install mariadb-server` - install MariaDB
2. systemctl to start and enable
3. `sudo firewall-cmd --add-service=mysql` 
4. `mysql_secure_installation` - script to increase security in database
5. `/etc/my.cnf.d/mariadb-server.cnf`:
```
datadir=
socket=
log-error=
pid-file=/run/mariadb/mariadb.pid # pid of database's process
```

### 13. Manage and configure Virtual Machines
QEMU/KVM is a virtualization solution on Linux (main one). QEMU/KVM can managed by `virsh` command.
* `sudo dnf install libvirt qemu-kvm` to install 
    - `libvrt` - interact with vms
    - `qemu-kvm` - create them
* `virsh help` - check commands
 - `domain` - virtual machine
 - `virsh reset`
 - `virsh shutdown`
 - `virsh destroy` (forced power-off)
 - `virsh undefine` - remove but keep data
 - `virsh undefine --remove-all-storage`
 - `virsh autostart`
 - `virsh dominfo` - info on VM

 #### add vcpus
 1. `virsh setvcpus TestMachine01 2 --config --maximum` - first set max
 2. `virsh setvcpus TestMachine01 2 --config` - then assign

### 14. Create and Boot a Virtual Machine ✨
* `qemu-img` - inspect img file
    - virtual size -> `qemu-img resize` to resize
    - copy to `/var/lib/libvirt/images`
* `virt-install --osinfo ubuntu 22.04 --name ubuntu01 --memory 1024 --vcpus 1 --import --disk /var/lib/libvirt/images/ubuntu-22.04-minimal-cloudimg-amd64.img --graphics none` - example command for creating vm with ubuntu 
* `--cloud-init` - to utilize cloudinit, e.g. for root password creation 

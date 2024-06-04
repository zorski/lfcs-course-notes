## NETWORKING

### 1. Configure IPv4 and IPv6 networking and hostname resolution

To set ip address (temporary)
* `sudo ip address add <ipv4_ipv6_address>/<cidr> dev <device>`
* `sudo ip address delete <address> dev <device>`

To set ip address permanently (depends on distribution). Ubuntu uses `netplan`:
* `netplan get` - prints current config
* `/etc/netplan` - location for configuration files (`.yaml`)
* `sudo netplan apply` - to apply configuration
* `sudo netplan try` - to apply for a set time (after timeout changes will be reverted)

#### example netplan config"
```yaml
network:
  ethernets:
    enp0s8:
      dhcp4: false
      dhcp6: false
      addresses:
        - 10.0.0.9/24
        - fe80:921b:eff:fe3d:abcd/64
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      routes:
        - to: 192.168.0.0/24
          via: 10.0.0.100
        - to: default
          via: 10.0.0.1
version: 2
```
#### other exmaples
`ls /usr/share/doc/netplan/examples`


### 2. Start, stop, and check the status of network services
`ss` and `netstat` tools can be used to check to inspect what programs are running and waiting for incoming network connections.
* `sudo ss -tunlp` - to see programs waiting for incoming network connections
    - `-l` - listening
    - `-t` / `-u` - tcp / udp
    - `-n` - port numbers / numer values
    - `-p` - processes


### 3. Configure bridge and bonding devices
* **bond** is grouping of two or more interfaces into one, traffic is load-balanced, and one link can go down
 - it can make network more resilient
 - increase network throughput
* **bridge** is a connection between two interfaces and a result allowing to connect two networks (just like a bridge connects two masses of land...)

#### bonding modes:
* mode 0 - "round-robin"
* mode 1 - "active-backup"
* mode 2 - "XOR" / *source-dest pair will go through the same interface*
* mode 3 - "broadcast" / *all data is send through all network interfaces at once*
* mode 4 - "IEEE 802.3ad" / *can increase network throughput*
* mode 5 - "adaptive transmit load balancing" / *load balances on least busy*
* mode 6 - "adaptive load balancing" / *both outgoing and incoming LB*

### 4. Demo: Configure bridge and bonding devices
Demos with `netplan`.
* `man netplan` for bonding modes etc.

bridge:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s9:
      dhcp4: no
    enp0s10:
      dhcp4: no
  bridges:
    br0:
      dhcp4: yes
      interfaces:
        - enp0s9
        - enp0s10
```

bond:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s9:
      dhcp4: no
    enp0s10:
      dhcp4: no
  bonds:
    bond0:
      dhcp4: yes
      interfaces:
        - enp0s9
        - enp0s10
      parameters:
        mode: active-backup
        primary: enp0s9
```

### 5. Configure packet filtering
On Ubuntu `ufw` is used for pakcket filtering.
* `sudo ufw status`
* `sudo ufw allow 22/tcp` or `sudo ufw allow ssh`
* `sudo ufw enable`
* `sudo ufw status verbose`
* `sudo ufw status numbered`
* `sudo ufw delete 1` - rule number 
* `sudo ufw delete allow 22` - or my rule name
* `sudo ufw allow from 10.10.10.0/24 to any port 22`
* `sudo ufw insert 3 deny from 10.11.12.100` - insert rule at position and rest of rules will be moved down
* `sudo ufw deny out on enp0s3 to 8.8.8.8` 


### 6. Port Redirection and NAT (Network Address Translation)
Port forwarding and NAT (masquerading). 

On Ubuntu:
1. edit `/etc/sysctl.conf` or `/etc/sysctl.d/99-sysctl.conf` (preferred)
    - uncomment `net.ipv4.ip_forward=1` (`net.ipv6.conf.all.forwarding=1` for IPv6)
2. `sudo sysctl --system` to reload
3. `sudo sysctl -a | grep forward` to check if options are toggled on
4. `iptables` to set forwarding:
```sh
sudo iptables -t nat -A PREROUTING -i enp1s0 -s 10.0.0.0/24 -p tcp --dport 8080 -j DNAT --to-destination 192.168.0.5:80

sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o enp6s0 -j MASQUERADE
```
5. `sudo apt install iptables-persistent` install package to persist rules
6. `sudo netfilter-persistent save` saves rules

`man ufw-framework`
`sudo iptables --list-rules --table nat` to list current rules
`sudo iptables --flush --table nat` to clear `nat`

### 7. Implement reverse proxies and load balancers

Sample config for nginx reversy proxy `/etc/nginx/sites-available/proxy.conf`:
```
server{
    listen 80;
    location / {
        proxy_pass http://1.1.1.1;
        include proxy_params; # includes headers
    }
}
```
Configuration files in `sites-available` directory are not served, to do make this configuration active configuration needs to be put in `sites-enabled` directory. The standard practice is soft-linking the `.conf` files from `sites-available` in `sites-enabled`:
```
sudo ln -s /etc/nginx/sites-available/proxy.conf /etc/nginx/sites-enabled/proxy.conf
sudo nginx -t # to check config
sudo nginx -s reload # hot reload
```
To disable default site: `sudo rm /etc/nginx/sites-enabled/default`

### Load Balancer
For load balancing functionality a group (collection) of servers can be specified in separate block and used further in the config, like below:
```
upstream mybackendservers{
    server 10.10.0.45;
    server 10.10.0.55;
}

server{
    listen 80;
    location / {
        proxy_pass http://mybackendservers;
    }
}
```
The default load balancing mechanism will be `round-robin`, but also other mechanisms can be used:
* least_conn - request is passed to the server with the least number of active connections
* least_time - request is passed to the server with the least average response time and least number of active connections

```
upstream mybackendservers{
    least_conn;
    server 10.10.0.45 weight=3
    server 10.10.0.55:8081 weight=2 down;
    server 10.10.20.35 backup;
}
```
More on parameters and whole upstream module: http://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream

### 8. Set and synchronize system time using time servers
`timedatectl` 
`timedatectl set-ntp true`
`systemctl status systemd-time`
`sudo vim /etc/systemd/timesyncd.conf` - conf file
`sudo systemctl restart systemd-timesyncd` - restart service
`timedatectl --show-timesync`



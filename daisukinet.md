# Recommend
```
vim ~/.vimrc  
set syntax on  
set number  
set tabstop=4
echo "alias ls='ls --color'" >> /etc/profile
echo "alias ..='cd ..'" >> /etc/profile
echo "alias c='clear'" >> /etc/profile
```
# WAN
### PPPoE Client
```
int gi0/0
    pppoe-client dial-pool-number 1
    no shut
int gi0/1
    ip add 192.168.1.254 255.255.255.0 //local
    ip nat inside
    ip tcp adjust-mss 1414
    no shut
int dialer1
    ip address negotiated
    ip mtu 1454
    ip nat outside
    no cdp enable
    encapsulation ppp
    dialer pool 1
    //dialer-group 1
    ppp authentication chap callin
    ppp chap hostname skills
    ppp chap password skills
ip route 0.0.0.0 0.0.0.0 dialer 1
ip nat inside source list 1 interface dialer 1 overload
access-list 1 permit 192.168.1.0 0.0.0.255 //local
//dialer-list 1 protocol ip permit
```
### IPsecVPN
```
crypto isakmp policy 1
    encryption 3des
    hash md5
    authentication pre-share
    group 2
crypto isakmp key cisco address 200.1.1.1 //remote
crypto isakmp keepalive 30 periodic
crypto ipsec transform-set IPSEC esp-3des esp-md5-hmac
crypto map M-ipsec 1 ipsec-isakmp
    set peer 200.1.1.1 //remote
    set transform-set IPSEC
    match address A-ipsec
//int loopback 1
    //ip add 100.1.1.1 255.255.255.255 //local
int gi0/0
    pppoe enable group global
int dialer 1
    crypto map M-ipsec
ip access-list extended A-ipsec
    permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 //local remote
```
# Samba
### Server
```
apt -y install samba
```
vim /etc/samba/smb.conf
```
[Home]
hosts allow = 192.168.1.0/24
path = /home/user/samaba
#guest ok = yes
writable = yes
valid users = @share_group
```
```
useradd -m user
passwd user
smbpasswd -a user
groupadd share_group
gpasswd -a user share_group
mkdir /home/user/samaba
chgrp share_group
chmod 770 /home/user/samaba
```
### Client
```
apt -y install cifs-utils
mkdir ~/test
echo "mount.cifs //192.168.1.11/home ~/test -o username=user,password=Pass">> /etc/profile
source /etc/profile
```
# Load balancer
```
apt -y install haproxy
```
    


vim /etc/haproxy/haproxy.cfg
``` 
frontend web_proxy_http
default_backend web_servers_http
bind *:80

frontend web_proxy_https
default_backend web_servers_https
bind *:443

backend web_servers_http
server web01 192.168.1.1:80
server web02 192.168.1.2:80

backend web_servers_https
server web01 192.168.1.1:443
server web02 192.168.1.2:443
```
# CA
```
cd /ca/private
openssl genrsa -aes128 2048 > server.key
openssl req -new -key server.key > server.csr
openssl x509 -in server.csr -days 365000 -req -signkey server.key > server.crt
chmod 200 server.key
```
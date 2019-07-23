# Recommend
vim ~/.vimrc  
```
set syntax on  
set number  
set tabstop=4
```
```
echo "alias mkdir='mkdir -p'" >> /etc/profile
echo "alias rm='rm -r'" >> /etc/profile
echo "alias ls='ls --color'" >> /etc/profile
echo "alias ..='cd ..'" >> /etc/profile
echo "alias c='clear'" >> /etc/profile
```
# WAN
## PPPoE Client
```
int gi0/0
    pppoe-client dial-pool-number 1
int gi0/1
    ip add 192.168.1.254 255.255.255.0
    ip nat inside
    ip tcp adjust-mss 1414
int dialer 1
    ip add nego
    ip mtu 1454
    ip nat outside
    enc ppp
    dialer pool 1
    dialer-g 1
    ppp auth chap call
    ppp chap host test1@example.com
    ppp chap pass Cisco123
ip route 0.0.0.0 0.0.0.0 dialer 1
ip nat inside source list 1 int dialer 1 overload
access-list 1 permit 192.168.1.0 0.0.0.255
dialer-list 1 protocol ip permit
```
## IPsecVPN With PPPoE
```
crypto isakmp policy 1
    encry 3des
    hash md5
    authentication pre-share
    group 2
crypto isakmp key cisco address 200.1.1.1
crypto isakmp keepalive 30 periodic
crypto ipsec transform-set IPSEC esp-3des esp-md5-hmac
crypto map M-ipsec 1 ipsec-isakmp
    set peer 200.1.1.1
    set transform-set IPSEC
    match address A-ipsec
interface Loopback1
    ip address 100.1.1.1 255.255.255.255
interface GigabitEthernet 0/0
    pppoe enable group global
    pppoe-client dial-pool-number 1
    no cdp enable
interface GigabitEthernet0/1
    ip address 192.168.1.254 255.255.255.0
    ip tcp adjust-mss 1356
interface Dialer1
    ip unnumbered Loopback1
    ip access-group A-security in
    ip mtu 1454
    encapsulation ppp
    dialer pool 1
    dialer-group 1
    no cdp enable
    ppp authentication chap callin
    ppp chap hostname cisco@cisco.com
    ppp chap password cisco
    crypto map M-ipsec
ip route 0.0.0.0 0.0.0.0 Dialer1
ip access-list extended A-ipsec
    permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
ip access-list extended A-security
    permit esp host 200.1.1.1 host 100.1.1.1
    permit udp host 200.1.1.1 host 100.1.1.1 eq isakmp
dialer-list 1 protocol ip permit
```
# Samba
## Server
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
## Client
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
# Bind9
```
apt -y install bind9
```
## Common
vim /etc/bind9/named.conf.options
```
dnssec-validation no;//検証しない
```
## lb1
vim /etc/bind9/named.conf.options
```
forwarders{200.99.1.1;};//回送
allow-recursion{localnet};//再帰問い合わせ
allow-transfer{200.99.1.1;DCIN};//ゾーン転送
```
vim /etc/bind9/named.conf.default-zones
```
zone "netad.it.jp" {
    type master;
    file "/etc/bind/db.netad.it.jp";
    allow-transfer{SLAVE-IP;};
    notify yes;
}
```
vim /etc/bind9/db.netad.it.jp
```
Add records by the case...
EX:
    sv1.example.org. IN A 192.168.1.1
    @ IN NS sv1.example.org // A server which manages example.org is sv1.example.org.
    @ IN A 192.168.1.1 // @.example.org is example.org
    www IN A 192.168.1.1 // www.example.org is 192.168.1.1
    local-www IN CNAME www.example.org. // 別名
    1.1.168.192.in-addr.arpa. IN PTR sv1.example.org. // 逆引き
    mail IN MAX 10 sv1.example.org. //10 is priority
```
## dcin
vim /etc/bind9/named.conf.options
```
forwarders{lb1-IP;};//回送
```
vim /etc/bind9/named.conf.default-zones
```
zone "netad.it.jp" {
    type slave;
    masters{lb1-IP;};
}
```
# Switch
## IP & GW
```
interface vlan 1
    ip add 192.168.1.1 255.255.255.0
ip default-gateway 192.168.1.254
```
## Etherchannel
```
interface range gi 1/0/2 - 3 //0は固定1スタックの２から３まで
    switchport trunkencapsulation dot1q
    switchport mode trunk
    channel-group 1 mode on
```
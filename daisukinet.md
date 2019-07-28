# wakaran
man bind9  
ospfパケット流す範囲制限  
バイト　開放日
# Recommend
vim ~/.vimrc  
```
syntax on  
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
## Tunnel With IPsecVPN
```
crypto isakmp policy 1
    encry 3des
    hash md5
    authentication pre-share
    group 2
crypto isakmp key cisco address 200.1.1.1
crypto isakmp keepalive 30 periodic
crypto ipsec transform-set IPSEC esp-3des esp-md5-hmac
crypto ipsec profile VTI
    set transform-set IPSEC
interface tunnel 0
    ip add 192.168.100.1 255.255.255.0
    tunnel source gi0/0
    tunnel destination 200.1.1.1
    tunnel mode ipsec ipv4
    tunnel protection ipsec profile VTI
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
chgrp share_group /home/user/samaba
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
## 学び
```
recursion = 再帰
forwardがうまくいかないときはallow-recursionをチェック
```
![再帰　反復　違い　イメージ](https://go-journey.club/wp-content/uploads/2017/04/SnapCrab_NoName_2017-4-15_16-44-4_No-00.png)  
参考URL : [DNSフォワーディング](https://go-journey.club/archives/1430)
```
apt -y install bind9
```
## Common
vim /etc/bind/named.conf.options
```
dnssec-validation no;//検証しない
```
## lb1
vim /etc/bind/named.conf.options
```
forwarders{200.99.1.1;};//回送
allow-recursion{20.0.0.0/28;};//再帰問い合わせ
allow-transfer{200.99.1.1;DCIN};//ゾーン転送
```
vim /etc/bind/named.conf.default-zones
```
zone "netad.it.jp" {
    type master;
    file "/etc/bind/db.netad.it.jp";
    notify yes;
}
```
vim /etc/bind/db.netad.it.jp
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
vim /etc/bind/named.conf.options
```
forwarders{20.0.0.1;};//回送
```
vim /etc/bind/named.conf.default-zones
```
zone "netad.it.jp" {
    type slave;
    masters{20.0.0.1;};
}
```
# Switch
## IP & GW
```
interface vlan 1
    ip add 192.168.1.1 255.255.255.0
ip default-gateway 192.168.1.254
```
# BFD & BGP
```
int gi0/0
    ip add 200.100.10.2
    bfd interval 50 min_rx 50 multiplier 3
router bgp 100
    neighbor 200.100.10.2 fall-over bfd
router ospf process bangou
    network 200.100.10.2 255.255.255.252 area bagou
    bfd all interfaces
```
# ルーター初期化
```
alt + b
confreg 0x2142 // rom読み込まない
reset
```
```
config-register 0x2102 //rom読み込む　だから再起動時にwriteが反映される
```
# SCP
## server
```
apt install ssh
touch /home/master/ca
```
## client
```
scp master@serverip:/home/master/ca ~/pass
```
# キーボード変更
## 英語キーボード
/etc/default/keyboard
```
XKBMODEL="pc105"
XKBLAYOUT="us"
```
## 日本語キーボード
/etc/default/keyboard
```
XKBMODEL="jp106"
XKBLAYOUT="jp"
```
# LVM
```
fdisk -l //現在のディスクの状況確認
fdisk /dev/sdb
n //new
p //primary
8e //file system type "LVM"
w //write
fdisk -l /dev/sdb //パーティションが作成されたことを確認
pvcreate /dev/sdb1 /dev/sdc1　//物理ボリュームの作成
pvdisplay　//確認
vgcreate lvg-share /dev/sdb1 /dev/sdc1　//lvg-share ボリュームグループを作成
vgdisplay　//確認
lvcreate --name public --size 18GB lvg-share　//論理ボリュームを作成
lvdisplay //確認
mkfs.ext4 /dev/lvg-share/public //public論理デバイスにext4ファイルシステムを作成する
sudo mount /dev/lvg-share/public /mnt/public >> /etc/profile //マウント
source /etc/profile
```
# コマンド一覧を表示するのコマンド
compgen -ac //-a aliasも表示 -cコマンドを表示？
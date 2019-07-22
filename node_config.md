# Ciscoネットワークノード設定課題

##  便利設定
- コマンド誤入力によるDNS検索を行わない。  
```Router(config)#no ip domain-lookup```

- コマンド履歴バッファの保存数を100行に設定する。  
```Router#terminal history size 100```

- ルータ上のHTTPサーバは停止する。  
```Router(config)#no ip http server```

- 日時を手動設定する。競技場との誤差は５分以内にする。  
```Router#clock set 21:10:00 14 June 2019```

- コンソール接続時、自動ログアウト機能を無効にする。  
    ```
    Router(config)#line console 0
    Router(config-line)#exec-timeout 0 0
    or
    Router(config-line)#no exec-timeout
    ```
    
- コンソール接続時、表示割込みに対する入力文字列の保管を有効にする。  
    ```
    Router(config)#line console 0
    Router(config-line)#logging synchronous
    ```

- コンソール接続時、--More--機能を無効にする。  
    ```
    Router(config)#line console 0
    Router(config-line)#length 0
    ```

- telnet 接続時、５分操作が行われなかった場合、自動ログアウトする。  
    ```
    Router(config)#line vty 0 4
    Router(config-line)#exec-timeout 5 0
    ```
##  全ネットワークノード共通設定　　

- タイムゾーンを日本標準時に設定する。  
```Router(config)#clock timezone JST 9```
- コンソール接続に対するパスワードは設定しない。
- イネーブルパスワードは"cisco"とする。イネーブルパスワードは暗号化すること。  
```Router(config)#enable secret cisco```

## ルーティング設定
- RORTのデフォルトルート(Dialer0)を静的に設定する。  
```RORT(config)#ip route 0.0.0.0 0.0.0.0 Dialer 0```
- DCRT1 および DCRT2 において次の通り BGP を動作させる。※ISP-A は AS200、ISP-B は AS300、ISP-C は AS600 所属として BGP が動作している。
	- DCRT1 と DCRT2 は AS100 所属として BGP を動作させる。
	
	```
	DCRT1(config)#router bgp 100
	```  
	```
	DCRT2(config)#router bgp 100
	```
	
	- DCRT1 は ISP-A と eBGP ピアを確立する。
	```DCRT1(config-router)#neighbor 200.100.10.2 remote-as 200```
	- DCRT2 は ISP-B と eBGP ピアを確立する。	
	```DCRT2(config-router)#neighbor 200.100.20.2 remote-as 300```
	- インターネット側から 20.0.0.0/28 宛てのトラフィックについて、ISP-B→DCRT2 を経由する 経路を優先経路とする。この経路に障害が発生した場合は、ISP-A→DCRT1 を経由する経路に 自律的に切り替わること。
		
		```
		DCRT1(config-router)#network 20.0.0.0 mask 255.255.255.240
		DCRT1(config-rputer)#neighbor 200.100.10.2 weight 100
		```
		
		```
    	DCRT2(config-router)#network 20.0.0.0 mask 255.255.255.240
        DCRT2(config-router)#neighbor 200.100.10.2 weight 200
        ```
		
- RORT,DCRT1,DCRT2 において次の通り OSPF を動作させる
	- プライベートアドレスセグメントについて経路交換を行う。
	
	```
	DCRT1(config)#router ospf 1
	DCRT1(config-router)#network　172.16.1.0 0.0.0.255 area 0
	DCRT1(config-router)#network 20.0.0.0 0.0.0.15 area 0
	DCRT1(config-router)#network 192.168.101.0 0.0.0.255 area 0
    ```
    
	```
    DCRT2(config)#router ospf 1
    DCRT2(config-router)#network　172.16.1.0 0.0.0.255 area 0
    DCRT2(config-router)#network 20.0.0.0 0.0.0.15 area 0
    DCRT2(config-router)#network 192.168.101.0 0.0.0.255 area 0
    ```
    
	```
	RORT(config)#router ospf 1
    RORT(config-router)#network　172.16.1.0 0.0.0.255 area 0
    RORT(config-router)#network 20.0.0.0 0.0.0.15 area 0
    RORT(config-router)#network 192.168.101.0 0.0.0.255 area 0
    ```

	- インターネット側（トンネル回線除く）と ROSW 側へ OSPF 経路情報を流さないこと。

## WAN 設定
- RORT にて PPPoE クライアント設定を行い ISP-C（PPPoE サーバ設定済み）と接続する。
※ISP-C から IPCP によって固定アドレスが払い出される。
    - Dialer インタフェースとして Dialer0 を作成する。
    - 認証方式は chap とし、認証ユーザは“skills”、パスワードは“skills”を用いる。
    ```
    !
    interface GigabitEthernet 0/0
    no ip address
    pppoe enable
    pppoe-client dial-pool-number 1
    no shutdown
    !
    interface GigabitEthernet 0/1
    ip address 172.16.1.254 255.255.255.0
    ip nat inside
    ip tcp adjust-mss 1414
    !
    interface Dialer 0
    ip address negotiated
    ip mtu 1454
    ip nat outside
    encapsulation ppp
    dialer pool 1
    dialer-group 1
    ppp authentication chap capllin
    ppp chap hostname skills
    ppp chap password skills
    !
    ip route 0.0.0.0 0.0.0.0 Dialer 0
    !
    ip nat inside source list 1 interface Dialer 0 overload
    !
    access-list 1 permit 172.16.1.0 0.0.0.255
    dialer-list 1 protocol ip permit
    !
    ```
- DCRT1 と RORT を [IPsecVPN](https://www.infraexpert.com/study/ipsec13.html) 接続する。
    - DCRT1-RORT 間に 10.1.0.0/30（DCRT1 側が若番）のアドレスを使用したトンネルインターフェ ース Tunnel0 を作成し、[IPSec VTI](http://klock-3rd.hatenablog.com/entry/2015/04/18/180147)(Vitual Tunnel Interface)として設定する。

# ゲートウェイの冗長化  
DCRT1 と DCRT2 において、以下の条件を満足するようにゲートウェイの冗長構成を実現しなさい。
- VLAN20 について、[VRRP](https://www.infraexpert.com/study/fhrpz07.html) を次の通り動作させる。
	- DCRT2 を Master ルータとする。
```
DCRT2(config) # interface vlan 20
DCRT2(config-if) # ip address 192.169.101.254 255.255.255.0
DCRT2(config-if) # vrrp 20 ip 192.169.101.254
```
```
DCRT1(config) # interface vlan 20
DCRT1(config-if) # ip address  192.169.101.253 255.255.255.0
DCRT1(config-if) # vrrp 20 ip 192.168.101.254
```

- VLAN101 について、[HSRP](https://www.infraexpert.com/study/fhrpz07.html) を次の通り動作させる。
	- DCRT1 を Active ルータとする。
```
DCRT1(config) # interface vlan 101
DCRT1(config-if) # ip address 20.0.0.14 255.255.255.240
DCRT1(config-if) # standby 101 ip 20.0.0.14
DCRT1(config-if) # standby 101 priority 105
DCRT1(config-if) # standby 101 preempt
```
```
DCRT2(config) # interface vlan 101
DCRT2(config-if) # ip address 20.0.0.13 255.255.255.0
DCRT2(config-if) # standby 101 ip 20.0.0.14
DCRT2(config-if) # standby 101 priority 100
```

# NAPT
アドレス変換を以下の通り設定しなさい。
- ROLAN(172.16.1.0/24)からインターネット（想定）へ接続できるように RORT に NAPT を設定しなさい。使用するグローバルアドレスは Dialer0 に設定されているアドレスとする。

# スイッチ設定
スイッチについて以下の通り各種設定を行いなさい。  
- 各スイッチ（DCSW1, DCSW2）について、管理アドレスおよび適切なデフォルトルートを設定する。
- DCSW1 と DCSW2 間の接続について、[Etherchannel](https://www.infraexpert.com/study/etherchannelz4.html) を設定する。

# アクセスコントロール
アクセスコントロール RORT の Dialer0 での通信について、アクセスコントロールを以下の通り設定しなさい。[拡張ACL](https://www.infraexpert.com/study/aclz5.html)

- すべての送信元から RORT 自身に対する ICMP トラフィックを許可する。  
```access-list 100 permit icmp   host any Dialer 0```

- RORT と DCRT1 間の通信はすべて許可する。  
```
access-list 100 permit any 200.100.10.1 Dialer 0
access-list 100 permit any Dialer 0 200.100.10.1
```

- RORT と DCRT2 間の通信はすべて許可する。  
```
access-list 100 permit any 200.100.20.1 Dialer 0
access-list 100 permit any Dialer 0 200.100.20.1
```

- ROLAN からの NAPT された発信トラフィックとそれに対する戻りトラフィックを許可する。[再帰ACL](https://www.infraexpert.com/study/aclz13.html)?

```
Cisco(config)# ip access-list extened OUTBOUND
Cisco(config-ext-nacl)# permit icmp any any
Cisco(config-ext-nacl)# permit tcp any any　reflect TCPACL timeout 120
```

```
Cisco(config)# ip access-list extened INBOUND
Cisco(config-ext-nacl)# permit icmp any any
Cisco(config-ext-nacl)# evaluate TCPACL
```

```
Cisco(config)# interface Dialer 0
Cisco(config-if)# ip access-group OUTBOUND out 
Cisco(config-if)# ip access-group INBOUND in
```
- 上記以外は許可しない。

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
	```DCRT1(config)#router bgp 100```
    ```DCRT2(config)#router bgp 100```
	- DCRT1 は ISP-A と eBGP ピアを確立する。
	```
    DCRT1(config-router)#neighbor 200.100.10.2 remote-as 200
    ```
	- DCRT2 は ISP-B と eBGP ピアを確立する。	
	```
    DCRT2(config-router)#neighbor 200.100.20.2 remote-as 300
    ```
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

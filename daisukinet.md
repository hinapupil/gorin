疑問
mss サイズどうする問題

te

WAN設定
    PPPoEクライアント設定
        int gi0/0
            pppoe-client dial-pool-number 1
            no shut
        int gi0/1
            ip add 192.168.1.254 255.255.255.0 //ローカル側
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
        access-list 1 permit 192.168.1.0 0.0.0.255 //ローカル側
        //dialer-list 1 protocol ip permit
    IPsecVPN
        crypto isakmp policy 1
            encryption 3des
            hash md5
            authentication pre-share
            group 2
        crypto isakmp key cisco address 200.1.1.1 //相手側
        crypto isakmp keepalive 30 periodic
        crypto ipsec transform-set IPSEC esp-3des esp-md5-hmac
        crypto map M-ipsec 1 ipsec-isakmp
            set peer 200.1.1.1 //相手側
            set transform-set IPSEC
            match address A-ipsec
        //int loopback 1
            //ip add 100.1.1.1 255.255.255.255 //自分側
        int gi0/0
            pppoe enable group global
        int dialer 1
            crypto map M-ipsec
        ip access-list extended A-ipsec
            permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 //自分側　相手側

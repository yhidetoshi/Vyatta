![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyos-icon.png)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-icon2.png)

(※)VagrantでVyattaを作る場合は参考にこちら:-> https://github.com/yhidetoshi/Vagrant

※(以下、勉強中/検証のため、間違いがあるかもしれないです..)

Vyattaコマンドメモ　
=======
|コマンド    |説明         |
|:-----------|:------------|
|$ configure|コンフィグモードへ移行|
|# exit|オペレーションモードへ以降|
|# commit|設定適用|
|# save|設定保存|
|$ reboot|再起動|
|$ show version|バージョン確認|
|# set interfaces ethernet eth1 address 10.0.2.2/24|eth1に10.0.2.2/24のアドレスを付与する例|
|$ show interfaces|インターフェース確認|
|# set interfaces ethernet eth0 disable|インターフェースを無効化|
|# delete interfaces ethernet eth0 disable|インターフェースを有効化|
|# set system gateway-address ip_address|デフォルトゲートウェイを設定|
|$ show ip route|ルーティングを確認|
|$ show vrrp|vrrpの状態確認|

#### **[スタティックルート]**
-> Vagrantで4台のVyattaを構築
```
192.168.100.111/24           192.168.100.101/24        192.168.200.101/24           192.168.200.222/24
      |vyatta1|●----------------●|vyatta2|●----------------●|vyatta3|●----------------●|vyatta4|
                              172.16.100.101/24         172.16.100.102/24             
```

vyatta1からvyatta2(右側)のethに通信できるようにスタティックルートを設定
```
//4台ともeth0によって通信してしまうのでinterfaceを無効化
vagrant@vyatta-client1# set interfaces ethernet eth0 disable


//vyatta1からvyatta2へスタティックルートを作成
vagrant@vyatta-client1# set protocols static route  192.168.100.0/24 next-hop 172.16.100.101

vagrant@vyatta-client1:~$ ping  172.16.100.101
connect: Network is unreachable

//vyatta1のデフォルトゲートウェイを設定
# set system gateway-address 192.168.100.101

# ping 172.16.100.101
PING 172.16.100.101 (172.16.100.101) 56(84) bytes of data.
64 bytes from 172.16.100.101: icmp_req=1 ttl=64 time=0.451 ms

-> 疎通確認OK
//vyatta4にも同様に設定をして vyatta1===> vyatta4の疎通確認
$ traceroute 192.168.200.222
traceroute to 192.168.200.222 (192.168.200.222), 30 hops max, 60 byte packets
 1  192.168.100.101 (192.168.100.101)  0.433 ms  0.383 ms  0.297 ms
 2  172.16.100.102 (172.16.100.102)  1.751 ms  1.627 ms  1.497 ms
 3  192.168.200.222 (192.168.200.222)  3.779 ms  3.714 ms  3.644 ms
```

#### **[OSPF]**

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-network-v1.png)


|vyatta        |eth1        |LAN         |eth2        |LAN         |eth3        |LAN         |
|:-------------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|
|vyatta2       |192.168.1.11|192.168.1.0/24|10.0.2.11 | 10.0.2.0/24| 10.0.0.11  |10.0.0.0/24 |
|vyatta3       |10.0.2.2    | 10.0.2.0/24| 10.0.1.3   |10.0.1.0/24 |            |            |
|vyatta4       |10.0.0.2    | 10.0.0.0/24| 10.0.1.2   |10.0.1.0/24 |            |            |

```
[vyatta2]
# configure
# set protocols ospf parameters router-id 127.0.0.1
# set protocols ospf area 0.0.0.0 network 10.0.2.0/24
# set protocols ospf area 0.0.0.0 network 10.0.0.0/24
# set protocols ospf redistribute connected
# commit
# save

[vyatta3]
# configure
# set protocols ospf parameters router-id 127.0.0.2
# set protocols ospf area 0.0.0.0 network 10.0.2.0/24
# set protocols ospf area 0.0.0.0 network 10.0.1.0/24
# set protocols ospf redistribute connected
# commit
# save

[vyatta4]
# configure
# set protocols ospf parameters router-id 127.0.0.3
# set protocols ospf area 0.0.0.0 network 10.0.0.0/24
# set protocols ospf area 0.0.0.0 network 10.0.1.0/24
# set protocols ospf redistribute connected
# commit
# save
```
**(OSPF確認)**
```
[vyatta2]
$ show ip ospf neighbor

    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
127.0.0.2         1 Full/Backup       30.289s 10.0.2.2        eth2:10.0.2.11           0     0     0
127.0.0.3         1 Full/Backup       38.712s 10.0.0.2        eth3:10.0.0.11           0     0     0

[vyatta3]
$ show ip ospf neighbor

    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
127.0.0.1         1 Full/DR           32.462s 10.0.2.11       eth1:10.0.2.2            0     0     0
127.0.0.3         1 Full/Backup       39.589s 10.0.1.2        eth2:10.0.1.3            0     0     0

[vyatta4]
$ show ip ospf neighbor

    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
127.0.0.1         1 Full/DR           35.924s 10.0.0.11       eth1:10.0.0.2            0     0     0
127.0.0.2         1 Full/DR           34.679s 10.0.1.3        eth2:10.0.1.2            0     0     0
=========
```
- **さらに1台の[vyattta1]を追加**


![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-network-add.png)
```
[vyatta1]
# set protocols ospf parameters router-id 127.0.0.5
# set protocols ospf area 0.0.0.0 network 192.168.1.0/24
# set protocols ospf redistribute connected
```
|vyatta        |eth1        |LAN         |eth2        |LAN         |eth3        |LAN         |
|:-------------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|
|vyatta1       |192.168.1.2 |192.168.1.0/24|          |            |            |            |
|vyatta2       |192.168.1.11|192.168.1.0/24|10.0.2.11 | 10.0.2.0/24| 10.0.0.11  |10.0.0.0/24 |
|vyatta3       |10.0.2.2    | 10.0.2.0/24| 10.0.1.3   |10.0.1.0/24 |            |            |
|vyatta4       |10.0.0.2    | 10.0.0.0/24| 10.0.1.2   |10.0.1.0/24 |            |            |

(結果) vyatta1からvyatta4に疎通できるようになった。
```
$ ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_req=1 ttl=63 time=0.825 ms
```

**(vyatta2のeth3のコストを50、それ以外は全て10)にしてルートが変わるか確認**
[コストのつけ方]
```
# set interfaces ethernet eth3 ip ospf cost 50
```
```
(コスト変更前)
$ traceroute 10.0.1.2
traceroute to 10.0.1.2 (10.0.1.2), 30 hops max, 60 byte packets
 1  192.168.1.11 (192.168.1.11)  1.570 ms  1.347 ms  1.267 ms
 2  10.0.1.2 (10.0.1.2)  2.844 ms  2.765 ms  2.454 ms

(経路)#=> vyatta2 -> vyatta4

(コスト変更後)
$ traceroute 10.0.1.2
traceroute to 10.0.1.2 (10.0.1.2), 30 hops max, 60 byte packets
 1  192.168.1.11 (192.168.1.11)  0.411 ms  0.284 ms  0.178 ms
 2  10.0.2.2 (10.0.2.2)  1.585 ms  1.533 ms  1.468 ms
 3  10.0.1.2 (10.0.1.2)  2.341 ms  2.281 ms  2.209 ms

(経路)#=> vyatta2 -> vyatta3 -> vyatta4
```
※4台のVyattaのconfig:https://github.com/yhidetoshi/Vyatta/tree/master/OSPF-conf

### VLAN
#### タグVLAN

- **[vyatta2]<===>[vyatta4]**

(設定)
```
-> タグVLAN(id=100)で通信させる.以下のように設定
-> タグVLAN(id=50)で通信させる.(id=100と同様にvyatta3とvyatta4を設定)
```

[vyatta2]
```
#=>設定コマンド
# set interfaces ethernet eth2 vif 100
# set interfaces ethernet eth2 vif 100 address 10.0.2.101/24

#=>eth3のコンフィグ
ethernet eth2 {
        vif 100 {
            address 10.0.2.101/24
        }
    }
```

[vyatta3]
```
#=>設定コマンド
set interfaces ethernet eth1 vif 100
set interfaces ethernet eth1 vif 100 address 10.0.2.102/24

#=>eth1のコンフィグ
ethernet eth1 {
        vif 100 {
            address 10.0.2.102/24
        }
    }
```

- **[VLANで通信がきているか確認]**
```
# sudo tcpdump -ne -i eth1

07:12:21.194095 08:00:27:e9:fa:92 > 08:00:27:1b:a8:34, ethertype 802.1Q (0x8100), length 102: vlan 50, p 0, ethertype IPv4, 10.0.1.2 > 10.0.2.101: ICMP echo reply, id 25329, seq 5, length 64
```

[VRRP+OSPF]
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-vrrp.png)

|vyatta        |eth1        |LAN         |eth2        |LAN         |eth3        |LAN         |
|:-------------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|
|vyatta1       |192.168.1.2 |192.168.1.0/24|          |            |            |            |
|vyatta2       |            |            |10.0.1.11   | 10.0.1.0/24| 10.0.2.11  |10.0.2.0/24 |
|vyatta3       |10.0.1.9    | 10.0.1.0/24| 10.0.2.9   |10.0.2.0/24 |            |            |
|vyatta4       |            |            | 10.0.1.8   |10.0.1.0/24 |            |            |

- **[通信の流れ]**
-> vyatta1からvyatta4にvip('10.0.2.10')を利用してルーティング(OSPF)
-> VRRPの優先度を変えてルーティングが変わるか確認

**[VRRPの設定]**
```
[vyatta3]
# set interfaces ethernet eth2 vrrp vrrp-group 10 virtual-address 10.0.2.10
# set interfaces ethernet eth2 vrrp vrrp-group 10 advertise-interval 1
# set interfaces ethernet eth2 vrrp vrrp-group 10 priority 255

[vyatta2]
# set interfaces ethernet eth3 vrrp vrrp-group 10 virtual-address 10.0.2.10
# set interfaces ethernet eth3 vrrp vrrp-group 10 advertise-interval 1
# set interfaces ethernet eth3 vrrp vrrp-group 10 priority 100

```
- **[vyatta1の設定]**

-> vyatta1のデフォルトゲートウェイを仮想IP(10.0.2.10)に設定


**参考:(configの抜粋)**
```
ethernet eth2 {
        address 10.0.2.9/24
        vrrp {
            vrrp-group 10 {
                advertise-interval 1
                priority 255
                virtual-address 10.0.2.10
            }
        }
    }
```
**(vrrpの状態確認)**
```
[vyatta3]
$ sh vrrp
                                 RFC        Addr   Last        Sync
Interface         Group  State   Compliant  Owner  Transition  Group
---------         -----  -----   ---------  -----  ----------  -----
eth2              10     MASTER  no         yes    1m50s       <none>


[vyatta2]
$ sh vrrp
                                 RFC        Addr   Last        Sync
Interface         Group  State   Compliant  Owner  Transition  Group
---------         -----  -----   ---------  -----  ----------  -----
eth3              10     BACKUP  no         no     49s         <none>


#=> priorityが高いvyatta3がMasterで低いvyatta2がBACKUPになっている
```


**(経路確認)**
```
$ traceroute 10.0.1.8
traceroute to 10.0.1.8 (10.0.1.8), 30 hops max, 60 byte packets
 1  10.0.2.9 (10.0.2.9)  0.307 ms  0.245 ms  0.265 ms
 2  10.0.1.8 (10.0.1.8)  0.734 ms  0.989 ms  0.551 ms

#=> priorityの高いvyatta3を経由してvyatta4に到達しているのを確認
```

**(priorityを変更してvrrpの状態確認)**

-> vyatta2の優先度を250, vyatta3の優先度を100に設定
```
[vyatta2]
$ sh vrrp
                                 RFC        Addr   Last        Sync
Interface         Group  State   Compliant  Owner  Transition  Group
---------         -----  -----   ---------  -----  ----------  -----
eth3              10     MASTER  no         no     11s         <none>


[vyatta3]
$ sh vrrp
                                 RFC        Addr   Last        Sync
Interface         Group  State   Compliant  Owner  Transition  Group
---------         -----  -----   ---------  -----  ----------  -----
eth2              10     BACKUP  no         no     34s         <none>
```
**(経路確認)**
```
$ traceroute 10.0.1.8
traceroute to 10.0.1.8 (10.0.1.8), 30 hops max, 60 byte packets
 1  10.0.2.11 (10.0.2.11)  0.470 ms  0.285 ms  1.506 ms
 2  10.0.1.8 (10.0.1.8)  1.813 ms  1.655 ms  1.611 ms

#=> priorityの高いvyatta2を経由してvyatta4に到達しているのを確認
```

**(Masterのvyatta2を落としたらvyatta3経由で到達するか確認)**
```
[vyatta2]
$ sh vrrp
                                 RFC        Addr   Last        Sync
Interface         Group  State   Compliant  Owner  Transition  Group
---------         -----  -----   ---------  -----  ----------  -----
eth3              10     FAULT   no         no     11s         <none>


$ sh vrrp
                                 RFC        Addr   Last        Sync
Interface         Group  State   Compliant  Owner  Transition  Group
---------         -----  -----   ---------  -----  ----------  -----
eth2              10     MASTER  no         no     1m57s       <none>

#=>Backupだったvyatta3が昇格してMasterへ


$ traceroute 10.0.1.8
traceroute to 10.0.1.8 (10.0.1.8), 30 hops max, 60 byte packets
 1  10.0.2.9 (10.0.2.9)  0.441 ms  0.295 ms  0.223 ms
 2  10.0.1.8 (10.0.1.8)  1.162 ms  1.095 ms  0.967 ms

#=>vyatta3を経由してvyatta4に到達しているのを確認
```
-> **冗長機能が効いている事を確認できた。**

#### Firewall
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-fw-icon.jpeg)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-firewall.png)

|vyatta        |eth1        |LAN         |eth2        |LAN         |eth3        |LAN         |
|:-------------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|
|vyatta1       |192.168.1.2 |192.168.1.0/24|10.0.2.2  |10.0.2.2/24 |            |            |
|vyatta2       |            |            |10.0.1.11   | 10.0.1.0/24| 10.0.2.11  |10.0.2.0/24 |
|vyatta3       |10.0.1.9    | 10.0.1.0/24|            |            |            |            |
|vyatta4       |            |            | 10.0.1.8   |10.0.1.0/24 |            |            |

[ステートレスファイアウォール]

vyatta2をFWとして設定してvyatta1からvyatta4へ通信させて確認する

すべてのトラッフィクを止める
```
# set firewall name Outside-In default-action drop

[インターフェースにfirewallを適用]
# set interfaces ethernet eth3 firewall in name Outside-In

※ -> in : stop outside to inside packets
  -> out: stop inside to outside packets
```

[vyatta2 firewallの設定]
```
# show firewall
 name Outside-In {
     default-action drop
 }
```

[接続テスト@vyatta2 ( OUT->IN )]
```
(設定前)
vagrant@vyatta-client1:~$ ping 10.0.1.8
PING 10.0.1.8 (10.0.1.8) 56(84) bytes of data.
64 bytes from 10.0.1.8: icmp_req=1 ttl=63 time=0.594 ms
64 bytes from 10.0.1.8: icmp_req=2 ttl=63 time=0.612 ms

(設定後)
$ ping 10.0.1.8
PING 10.0.1.8 (10.0.1.8) 56(84) bytes of data.
^C
--- 10.0.1.8 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2000ms

#=> pingがvyatta4へ届くなった
```

[接続テスト@vyatta2 ( IN -> OUT )]
```
# ping 10.0.1.8
PING 10.0.1.8 (10.0.1.8) 56(84) bytes of data.
64 bytes from 10.0.1.8: icmp_req=1 ttl=64 time=0.373 ms

#=> 設定通りinside to outsideは通過
```

**[icmpを通過させる]**

-> `set firewall all-ping 'enable'`
```
vagrant@vyatta-client2# sh firewall
 all-ping enable
 name Outside-In {
     default-action drop
 }
```
(pingテスト)
```
$ ping 10.0.1.8
PING 10.0.1.8 (10.0.1.8) 56(84) bytes of data.
64 bytes from 10.0.1.8: icmp_req=1 ttl=63 time=0.622 ms

#=>再びvyatta1からvyatta4へping通信が可能になった.
```
**[FWのポリシーを試すためにCentOSのVMを接続して検証]**

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-fw-fig2.png)


|vyatta        |eth1        |LAN         |eth2        |LAN         |eth3        |LAN         |
|:-------------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|
|vm1           |192.168.1.2 |192.168.1.0/24|          |            |            |            |
|vyatta2       |            |            |10.0.1.11   | 10.0.1.0/24| 10.0.2.11  |10.0.2.0/24 |
|vyatta3       |10.0.1.9    | 10.0.1.0/24|            |            |            |            |
|vm4           |10.0.1.8    | 10.0.1.0/24|10.0.2.9    |10.0.2.0/24              |            |            |

- vyatta2とvyatta3はOSPF
- (メモ)[vm1とvm4のGATEWAY設定]
```
# cat ifcfg-eth1 | grep GATEWAY
GATEWAY=192.168.1.11

(vm4)
# cat ifcfg-eth1 | grep GATEWAY
GATEWAY=10.0.1.9
```


- **その他設定メモ** 
 タイムゾーンの変更
```
$ show configuration
$ configure
# set system time-zone Asia/Tokyo
# show system time-zone
time-zone Asia/Tokyo
[edit]
```

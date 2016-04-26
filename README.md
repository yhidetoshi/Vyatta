![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/vyatta-icon.png)

(※)VagrantでVyattaを作る場合は参考にこちら:-> https://github.com/yhidetoshi/Vagrant

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
|$ show interfaces|インターフェース確認|
|# set interfaces ethernet eth0 disable|インターフェースを無効化|
|# set system gateway-address ip_address|デフォルトゲートウェイを設定|
|$ show ip route|ルーティングを確認|

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
図をお借りしてswitchとポートは表でマッピング.

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vyatta/switch3.png)


|vyatta        |eth1(F0/1)|LAN|eth2(F0/2)|LAN|
|:-------------|:-----------|:-----------|:-----------|:-----------|
|vyatta1(SW-A) |10.0.2.11   | 10.0.2.0/24| 10.0.0.1   |10.0.0.0/24 |
|vyatta2(SW-B) |10.0.2.2    | 10.0.2.0/24| 10.0.1.3   |10.0.1.0/24 |
|vyatta3(SW-C) |10.0.1.2    | 10.0.1.0/24| 10.0.0.2   |10.0.1.0/24 |

```
[vyatta1]
# configure
# set protocols ospf parameters router-id 127.0.0.1
# set protocols ospf area 0.0.0.0 network 10.0.2.0/24
# set protocols ospf area 0.0.0.0 network 10.0.0.0/24
set protocols ospf redistribute connected
# commit
# save

[vyatta2]
# configure
# set protocols ospf parameters router-id 127.0.0.2
# set protocols ospf area 0.0.0.0 network 10.0.2.0/24
# set protocols ospf area 0.0.0.0 network 10.0.1.0/24
set protocols ospf redistribute connected
# commit
# save

[vyatta3]
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
[vyatta1]
$ show ip ospf neighbor

    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
127.0.0.2         1 Full/Backup       30.289s 10.0.2.2        eth2:10.0.2.11           0     0     0
127.0.0.3         1 Full/Backup       38.712s 10.0.0.2        eth3:10.0.0.11           0     0     0

[vyatta2]
$ show ip ospf neighbor

    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
127.0.0.1         1 Full/DR           32.462s 10.0.2.11       eth1:10.0.2.2            0     0     0
127.0.0.3         1 Full/Backup       39.589s 10.0.1.2        eth2:10.0.1.3            0     0     0

[vyatta3]
$ show ip ospf neighbor

    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
127.0.0.1         1 Full/DR           35.924s 10.0.0.11       eth1:10.0.0.2            0     0     0
127.0.0.2         1 Full/DR           34.679s 10.0.1.3        eth2:10.0.1.2            0     0     0
=========
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

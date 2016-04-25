![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vagrant/vyatta-icon.png)

  ##### Vyattaコマンド
  
  
- **タイムゾーンの変更** 
```
$ show configuration
$ configure
# set system time-zone Asia/Tokyo
# show system time-zone
time-zone Asia/Tokyo
[edit]
```
(結果確認)
```  
$show configuration

   time-zone Asia/Tokyo
```


#### Vyattaコマンドメモ　
====
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
|$ show ip route|ルーティングを確認|

- **[スタティックルート]**
-> Vagrantで4台のVyattaを構築
```
192.168.100.111/24           192.168.100.101/24        192.168.200.101/24           192.168.200.222/24
      |vyatta1|●----------------●|vyatta2|●----------------●|vyatta3|●----------------●|vyatta4|
                              172.16.100.101/24         172.16.100.102/24             
```

vyatta1からvyatta2(右側)のethに通信できるようにスタティックルートを設定
```
//vyatta1でeth0によって通信してしまうのでinterfaceを無効化
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
```

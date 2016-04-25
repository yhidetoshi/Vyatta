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
|$ show ip route|ルーティングを確認|




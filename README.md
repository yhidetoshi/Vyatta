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

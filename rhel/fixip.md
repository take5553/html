# IP固定

## 環境

RHEL 8

## 手順

設定ファイルは`etc/sysconfig/network-scripts/ifcfg-eth0`

おそらく末尾の`eth0`は色々変わる。`ifcfg-`ぐらいまで打って`Tab`キーを押したら勝手に補完されると思う。

上記ファイルに以下を追記。

~~~
ONBOOT=yes
IPADDR=192.168.1.204
PREFIX=24
GATEWAY=192.168.1.1
DNS1=192.168.1.1
DNS2=8.8.8.8
~~~

これでOSを再起動。

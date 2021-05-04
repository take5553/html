# 12日目　今更ながらファイアーウォールの設定　（※失敗）

※上手くいかなかった。以下記録として。



とりあえず使うことを優先してきたが、割と不自由しないので、それならばと本格移行を考えてみたところ、セキュリティがガバガバなのでファイアーウォールを設定していく。

## 基礎知識

ファイアーウォールの設定といえば`iptables`（＝ポートの開け閉めを担当するソフト。正確には各ポートを通過しようとするパケットを監視して、ルールに沿って許可、拒否、無視というフィルタリングをするソフト。）の設定を指す。

`iptables`はいくつかの「テーブル」と呼ばれるものがあって、テーブルの中には「チェイン」というものがある。チェインは「ルール」のリスト。

今から主に編集していくのは`filter`と呼ばれるテーブルの中の、`INPUT`と呼ばれるチェイン。`filter`には他にも`OUTPUT`と`FORWARD`と呼ばれるチェインがある。

|       INPUTチェイン       |      OUTPUTチェイン       |                FORWARDチェイン                 |
| :-----------------------: | :-----------------------: | :--------------------------------------------: |
| 外<br />↓<br />中のソフト | 中のソフト<br />↓<br />外 | 外<br />↓<br />（中を全スルー）<br />↓<br />外 |

自分で立ち上げたソフトには通信を許可したいので`OUTPUT`は全OK、このPCはルーターみたいに中継することはないので`FORWARD`は全NGとする。

`INPUT`チェインは基本全NGだけど、必要なものだけ許可しないといけない。

## 設定

### 初期状態

立ち上がってすらいない。

~~~shell
$ sudo systemctl status iptables
○ iptables.service - IPv4 Packet Filtering Framework
Loaded: loaded (/usr/lib/systemd/system/iptables.service; disabled; vendor preset: disabled)
Active: inactive (dead)
~~~

なので立ち上げて、次回から自動で立ち上がるようにする。

~~~shell
$ sudo systemctl start iptables
$ sudo systemctl enable iptables
~~~

最初の状態を見てみる。

~~~shell
$ sudo iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target     prot opt in     out     source               destination
~~~

こう表示されるはず。これは何もルールが設定されていないということ。ガバガバ。

### `FORWARD`チェインを全部NGにする

この内`OUTPUT`チェインはガバガバのままでいいけど、`FORWARD`チェインは全部NGとしなければならない。

~~~shell
$ sudo iptables -P FORWARD DROP
~~~

もう一度見てみる。

~~~shell
$ sudo iptables -nvL
Chain INPUT (policy ACCEPT 13 packets, 815 bytes)
pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy DROP 0 packets, 0 bytes)
pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 7 packets, 547 bytes)
pkts bytes target     prot opt in     out     source               destination
~~~

真ん中の`FORWARD`チェインのポリシーが`DROP`に変わった。

### 変更は手動で保存

このままだと再起動したら元に戻る。変更した設定は保存しておかないといけない。

~~~shell
$ sudo iptables-save | sudo tee /etc/iptables/iptables.rules
~~~

`>`でリダイレクトさせても上手くいかない（`sudo`で実行しているから）。

今後も設定する度に、再起動するまでには設定を保存しておかないといけない。

参考：[シェルでファイルにリダイレクトしたときに”Permission denied”と怒られたときの対処 | ゲンゾウ用ポストイット](https://genzouw.com/entry/2019/06/23/102944/1617/)

### `INPUT`チェインの設定

#### ユーザー定義チェインの作成

`INPUT`チェインは色々書いてしまいがちなので、ユーザー定義チェイン`TCP`、`UDP`を作成しておく。

~~~shell
$ sudo iptables -N TCP
$ sudo iptables -N UDP
~~~

#### `INPUT`チェイン最初のルール

`INPUT`チェインの大前提として、すでに接続が確立しているもの(`ESTABLISHED`)や、接続失敗時の返事のような特殊なパケット(`RELATED`)は受け入れないといけない。

そのようなパケットを探すには、以下のように打つ。

~~~shell
$ sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
~~~

* `-A (チェイン名)`：チェインにルールを追加する

* `-m (マッチ方法)`：マッチさせる方法を明示的に指定する。（後で暗黙的指定もやる）

* `conntrack --ctstate (状態)`：`conntrack`というモジュールを使用してマッチするかどうか判断。この状態とは、やり取りされるパケットに対してカーネルが追跡をして割り当てるものらしい。

  参考：[iptablesのマッチ](https://straypenguin.winfield-net.com/ipttut/output/matches.html)、[ステート機構](https://straypenguin.winfield-net.com/ipttut/output/statemachine.html)

* `-j (別のチェイン、またはターゲット)`：別のチェインへジャンプするか、またはターゲット（パケットがすべての条件に一致したときのアクション）を実行する。`ACCEPT`、`DROP`は特別な組み込みターゲットでジャンプした瞬間にパケットが処理される。`REJECT`と`LOG`は拡張ターゲット（後述）。

#### localhostを使って自分へ投げ返すようなパケットも許可

そういうことをするソフトもあるんだとさ。

~~~shell
$ sudo iptables -A INPUT -i lo -j ACCEPT
~~~

* `-i (ネットワークインターフェイス)`：`eth0`とか`eth1`とか。`lo`はloopbackの略らしい。

#### `INVALID`状態のパケットは破棄

`INVALID`とは`conntrack`が付与する、不正な状態を意味する。

~~~shell
$ sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
~~~

#### ping要求は許可

呼ばれれば返事ぐらいはしましょう、ってことかな。

~~~shell
$ sudo iptables -A INPUT -p icmp --icmp-type 8 -m conntrack --ctstate NEW -j ACCEPT
~~~

* `-p (プロトコル名)`：`tcp`とか`udp`とか`icmp`とか。ICMPプロトコルはネットワーク上の機器の状態を診断するプロトコル。
* `icmp --icmp-type (番号)`：ICMPタイプ。8はエコー要求、0はエコー応答。エコーとは、ほぼpingと同義。

#### TCP、UDPプロトコルはそれぞれのチェインに飛ぶ

接続が確立したら`ESTABLISHED`として最初に書いたルールで処理されるから、ここでは接続し始めに対して反応するようにする。

~~~shell
$ sudo iptables -A INPUT -p udp -m conntrack --ctstate NEW -j UDP
$ sudo iptables -A INPUT -p tcp --syn -m conntrack --ctstate NEW -j TCP
~~~

#### TCP、UDPチェインのどれにも引っかからなかったパケットは拒否

~~~shell
$ sudo iptables -A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
$ sudo iptables -A INPUT -p tcp -j REJECT --reject-with tcp-reset
~~~

#### どれにも引っかからなかった他のプロトコルのパケットを拒否

~~~shell
$ sudo iptables -A INPUT -j REJECT --reject-with icmp-proto-unreachable
~~~

#### 最後にどれにも引っかからなかった謎パケットはすべて破棄

~~~shell
$ sudo iptables -P INPUT DROP
~~~

### 状態の確認

~~~shell
$ sudo iptables -nvL --line-numbers
Chain INPUT (policy DROP 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    44528   11M ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
2        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
3        0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate INVALID
4        0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8 ctstate NEW
5       56  7264 UDP        udp  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate NEW
6        0     0 TCP        tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp flags:0x17/0x02 ctstate NEW
7       30  3917 REJECT     udp  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable
8        0     0 REJECT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with tcp-reset
9        0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-proto-unreachable

Chain FORWARD (policy DROP 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 562 packets, 42953 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain TCP (1 references)
num   pkts bytes target     prot opt in     out     source               destination

Chain UDP (1 references)
num   pkts bytes target     prot opt in     out     source               destination
~~~

こんな状態になってたらOK。

## チェック

現状Garuda Linuxが外からパケットを受け付けるのはSambaのみ。（Barrierはクライアントなので、接続開始はGarudaが始点）

一度Sambaの接続を切ってからSambaを再起動して再接続しようとすると、このルールではブロックされるはず。

・・・と思ってやってみたら普通に繋がる。何やっても繋がる。一生懸命設定した`iptables`の設定を全部消して、全部`DROP`する設定にしても、Samba以外の接続は全部切れても、Sambaは繋がる。



Sambaは繋がる。



なんでやねん！

続く。
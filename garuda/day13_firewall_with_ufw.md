# 13日目　ファイアーウォールの設定再び

`iptables`はちゃんと設定をやり切るには難しいということなので、`iptables`を簡単に設定してくれるツールである`ufw`を使って設定する。

## 設定

最初からインストールはされているらしい。まずは`ufw`を立ち上げる。

~~~shell
$ sudo ufw enable
~~~

`ufw`は`uncomplicated firewall`（複雑でないファイアーウォール）のこと。基本的には`INPUT`を制御してくれる。

`OUTPUT`は全部OK、`FORWARD`は無効がデフォルト。

まずは`INPUT`をすべて拒否する。

~~~shell
$ sudo ufw default DENY
~~~

その後、Sambaに必要なポートを開ける。

~~~shell
$ sudo ufw allow 137:138/udp
$ sudo ufw allow 139,445/tcp
~~~

状態の確認。

~~~shell
$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
137:138/udp                ALLOW IN    Anywhere
139/tcp                    ALLOW IN    Anywhere
445/tcp                    ALLOW IN    Anywhere
137:138/udp (v6)           ALLOW IN    Anywhere (v6)
139/tcp (v6)               ALLOW IN    Anywhere (v6)
445/tcp (v6)               ALLOW IN    Anywhere (v6)
~~~

一生懸命設定しなくても、これでよかった。

ちなみに`iptables -nL`で設定を見てみたらすごいいっぱい設定が書き込まれている（長いので割愛）。

## テスト

今はSamba経由で`/home/takeshi/share`ディレクトリを共有できている。

~~~shell
$ sudo ufw delete allow 137:138/udp
$ sudo ufw delete allow 139/tcp
$ sudo ufw delete allow 445/tcp
~~~

これで設定したルールを削除できた。

~~~shell
$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip
~~~

この状態でSambaを再起動。

~~~shell
$ sudo systemctl restart smb nmb
~~~

そうするとWindowsからアクセスすることができなくなる。

再設定すると繋がる。これで想定どおりの動きになった。
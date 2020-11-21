# 起動前準備＆初期設定

立ち上げ方は2通りある。

- モニターやマウス・キーボードをつないでセットアップし、遠隔操作までできるようになってからモニター等を外す。（簡単）
- 最初からモニター・マウス・キーボード無しで男らしくセットアップしてしまう。（中級）

![img](image/startup/2020-08-16-01.png)

どちらもSDカードにOSを書き込むまでは同じ。前者はその後画面の指示に従いセットアップ。後者は「SSH許可、Wifi設定書き込み」「メインPCからネットワーク上のRaspberryPiを探す」「SSH接続」という段取りを踏む。リモートデスクトップ接続までいけばメインPC上でRaspberryPiのデスクトップ画面を開くことができ、モニターがあっても無くてもどっちでもよくなる。

## 購入

Raspberry Pi本体は、CPUとかメモリが取り付け済みのちっちゃいマザーボードと言った感じで、それだけだと「まじかよ」とつぶやきたくなるぐらいむき出し。以下の付属品が付いたスターターキットとか売ってるけど、恩恵があるのはぶっちゃけ電源のみ。

- 電源（必要。5V3Aのコンセント直取り。まあ探せば買える。スマホ充電ケーブルだとアンペア数が足りないかも）
- ケース（数あるケースの中から自分で選んで個性を出すべき。というかどうせ後から買い替えたくなる）
- microSDカード（大容量のものがゴミみたいな値段で売られてる。OSがプリインストール済みとか無駄に高いから自分でやるべき。OSアップデートが一時的に容量食うので、余裕をもった容量を選ぶ）
- HDMIケーブル（家に1本ぐらいあるでしょ）

そのほか、モニター・マウス・キーボードが「必要だったら」揃える。

## microSDにOS書き込み

メインPC上でMicroSDカードにOS（Raspberry Pi OS）を書き込んでおく。モニター有りか無しかで書き込み方法が変わる。どちらも[Raspberry Pi Downloads – Software for the Raspberry Pi](https://www.raspberrypi.org/downloads/)からDLできる。

### モニター有り・簡単・標準

上記リンクから「Raspberry Pi Imager for 」というリンクを辿れば、SDカードのフォーマットとOSの書き込みを同時にやってくれるソフトが手に入る。楽ちん。

### モニター無し

上記リンクから「Raspberry Pi OS (previously called Raspbian)」というリンクを辿り、「Raspberry Pi OS (32-bit) with desktop and recommended software」または「同おすすめソフト無し版」のどちらかのzipファイルをダウンロードする。中からイメージファイルが出てくるので、[イメージ書き込みソフト](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md)（balenaEtcher、Win32DiskImager、Upswift imgFlasherのどれか）を使ってmicroSDカードに書き込む。

イメージ書き込み後、microSDのトップに「ssh（拡張子無し）」という名前の中身のないファイルを作成し、さらに「wpa_supplicant.conf」という名前のファイルを、以下の中身で作成しておく。

```
country=JP
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
	ssid="SSID名"
	psk="暗号化キー"
}
```

これでRaspberryPiが勝手にWifiに繋ぎにいってくれる。SSHも許可しているので、メインPCからリモートでログインして残りの各種設定をする。

## 電源供給

モニター有りの場合は電源供給前にHDMIケーブルでモニターをつないでおくこと。モニター無しの場合SDカードアクセスランプが落ち着いたらOSが読み込まれたことになるので、その後で接続を試みるようにする。

## 設定

### モニター有り

パスワード決めろとか、言語とかタイムゾーンは何だとか、簡単なことを聞かれるので適当に設定。決めたパスワードはPiというユーザーのもの。Linuxサーバー的な話をするとrootユーザー以外にはPiというユーザーしか存在しない。

### モニター無し

まずRaspberryPiにどのIPが振られたのかを確認しないといけない。これはメインPCから総当たりでIPを探す方法と、ルーターの中を覗いてRaspberryPiに振られているIPを見つける方法がある。これはRaspberryPiが繋がっているWifiによる。ルーターが覗けるならそっちの方が早い。ホスト名が表示されるなら見つけるのは簡単だけど、もし表示されなくてもRaspberryPiのMacアドレスは「B8-27-EB-xx-xx-xx」か「DC-A6-32-xx-xx-xx」なので、それを手掛かりに特定する。もしルーターを覗くことが無理なら総当たりで探していくしかない。と言ってもそんな大変なものではなく、`nmap`というツールを使えば自動で総当たりpingを飛ばしてくれる。ダウンロード・使い方は[ここ](http://[ip address /- Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/remote-access/ip-address.md))の「nmap command」を読めば分かる。

IPが分かったら、メインPCからコンソールを立ち上げ

~~~shell
$ ssh pi@（見つけたIP）
~~~

などと打ちSSH接続を行う。初期パスワードは「raspberry」。ログインできたら

```shell
$ sudo raspi-config
```

と打ち、設定画面を立ち上げる。各種設定は[ここ](http://[RaspberryPi Raspbian ヘッドレスインストール（Buster編） /- Qiita](https://qiita.com/nori-dev-akg/items/38c2dfb108edb0d73908#初期設定))が参考になる。パスワードは絶対に変えておく。

## IP固定化

モニター有りの場合も、ここからはターミナルを立ち上げてコマンドラインで操作していくことになる。

### 前提知識：DHCPサーバー機能とアドレス範囲

通常、Wifiに繋ぎにいったらルーターが勝手にLAN内IPを振ってくれる（DHCPサーバー機能）。逆に言えば繋ぐまでどんなIPを振られるか分からない。これからはIPアドレスを使って遠隔でRaspberry Piを操作していくので、いちいち確認するのは面倒。なので固定をする。

割り振り用IPには範囲が必ず設定されている。その範囲外で固定すれば他の機器が邪魔することはないってこと。結局ここでルーターを覗くことになるけど、覗けない環境の場合でも自動割り振りの範囲を無視してIPを固定化することはできる。ただ他の機器のIPアドレスとぶつかっても知らんけどね。

普通は繋いだ順にIPが振られると思うので、Raspberry Piに適当に`192.168.0.50`とか割り当てとけば多分大丈夫。

ちなみにPC・ルーターと同じサブネット内で設定すること。例えばPCのIPが`192.168.1.2`で、デフォルトゲートウェイが`192.168.1.1`だったら、`192.168.1`の部分を同じにして、`192.168.1.50`という割り当てにすると同じサブネット内に設定できる。

ルーターを覗けばどこかにIPアドレスプールとかDHCPプールとかいう設定があるはず。それを確認する。もし変更できる権限があるのであれば、プール範囲を狭めてRaspberryPi用にIPを確保してもいいかもしれない。

### IP固定化作業（RaspberryPi内）

モニター有りの場合はターミナル（![image-20200922232752424](image/startup/image-20200922232752424.png)のマークをクリック）を立ち上げ、モニター無しの人はそのままSSH接続で、以下のコマンドを打つ。

※モニター有りでここまで来たけど、Linuxコマンドについては何も知らない人へ
下記コマンドの先頭の`$`マークは無視して`sudo nano ・・・`から打つ。詳しくは次回の記事で説明するが、とりあえずこれ以降`$`マークが付いているコマンドはすべてターミナルに打つものと覚えておいてほしい。

```shell
$ sudo nano /etc/dhcpcd.conf
```

「nano」というRaspberryPi付属のテキストエディタが立ち上がるので、以下の内容を最後に追加して`Ctrl + S`でセーブ、`Ctrl + X`でnanoを終了する。

```
interface wlan0
static ip_address=192.168.aaa.aaa/24
static routers=192.168.bbb.bbb
static domain_name_servers=192.168.bbb.bbb
```

`192.168.aaa.aaa`はRaspberryPiに割り当てたいIP、`192.168.bbb.bbb`はルーターのIP。

## OSアップデート

以下のコマンドを打つ。モニターがある場合は「RaspberryPiの設定」からアップデートしても同じ。結構な時間がかかる。終わったら一応再起動。

```shell
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo reboot
```

## メインPCからリモートデスクトップを試みる

### RaspberryPiにrdpを入れる

RDP（Remote Desktop Protocol）を入れるためにxrdpというソフトをインストールする。昔はVNCが必要だったけど今は要らないらしい。そこまで詳しくないのでよく分からない。

```shell
$ sudo apt-get -y install xrdp
```

### Windows標準のリモートデスクトップからアクセスを試みる

Windowsスタートボタンの右の「ここに入力して検索」欄に「リモートデスクトップ」と打ちエンターキーを押すと↓これが起動する。

![img](image/startup/image.png)

コンピューターにさっき決めたIP、ユーザー名はpi。

![img](image/startup/image-1.png)

画面タブでちょうどいいサイズを選んでおけば勝手にデスクトップサイズを合わせてくれる。

接続！

![img](image/startup/image-3.png)

キター！

今後同じようにメインPCからリモートデスクトップ接続をすればモニター・キーボード・マウス無しでRaspberry Piを操作できる。

ちなみにリモートデスクトップからはシャットダウンボタンを押してもシャットダウンしないので、ターミナルを立ち上げて以下のコマンドを打つ。

```shell
$ sudo shutdown -h now
```

するとシャットダウンが始まり、リモートデスクトップの接続が切れるので、ランプが落ち着いてから電源を抜く。

## 次回以降のための設定

Raspberry Piの設定画面を開く。

![image-20200923231637004](image/startup/image-20200923231637004.png)

Systemタブの設定。モニター無しの人は`$ sudo raspi-config`でそれっぽい設定を探す。

![image-20200923231740500](image/startup/image-20200923231740500.png)

Interfacesタブの設定。モニター無しの人はすでにSSHが有効化されているので無視。

![image-20200923231851525](image/startup/image-20200923231851525.png)

## 参考

[RaspberryPi Raspbian ヘッドレスインストール（Buster編） – Qiita](https://qiita.com/nori-dev-akg/items/38c2dfb108edb0d73908)
[Installing operating system images – Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)
[Setting up a Raspberry Pi headless – Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)
[IP Address – Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/remote-access/ip-address.md)
[Raspberry Pi 4 のMACアドレスの範囲が変わったぞ – Qiita](https://qiita.com/tomotomo/items/2ff445377c13f9db38e2)
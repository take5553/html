# 開発環境の整備

## VNCの設定

GUIの出来を確認する用。ただし、日本語配列でのキー入力が上手くいかなかった。

参考：
[Jetson Nanoにリモートデスクトップ(VNC)環境を用意する - Qiita](https://qiita.com/iwatake2222/items/a3bd8d0527dec431ef0f#%E6%96%B9%E6%B3%951-tigervnc%E3%82%92%E4%BD%BF%E3%81%86)
[TigerVNC - ArchWiki](https://wiki.archlinux.jp/index.php/TigerVNC#vncserver_.E3.81.AB.E6.8E.A5.E7.B6.9A.E3.81.99.E3.82.8B)
[Ubuntu 20 .04にVNC をインストールして構成 する方法 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-20-04-ja)
[ECSのVNCが英語キーボードで困った場合の回避方法 #2 | Alibaba Cloudの備忘ログ](https://www.bigriver.jp/?p=4243)

### Jetson Nano上

~~~shell
$ sudo apt update && sudo apt upgrade
$ sudo apt install tigervnc-standalone-server tigervnc-scraping-server
$ vncpasswd
(パスワードの設定をする)
$ x0vncserver -display :0 -passwordfile ~/.vnc/passwd
~~~

### ローカルPCから

設定方法は使用するVNCクライアントによって違う

- Windowsでは、[TightVNC](https://www.tightvnc.com/)、[RealVNC](https://www.realvnc.com/)、または [UltraVNC](https://www.uvnc.com/)。
- macOS では、組み込みの[画面共有](https://support.apple.com/guide/mac-help/screen-sharing-overview-mh14066/mac)プログラムを使用するか、[RealVNC](https://www.realvnc.com/) のようなクロスプラットフォームアプリ。
- Linuxでは 、 `vinagre` 、 `krdc` 、 [RealVNC](https://www.realvnc.com/) 、 [TightVNC](https://www.tightvnc.com/) など。

自分の場合は`krdc`がインストールされてたからそれを使った。

そして、クライアントからとにかく`(Jetson NanoのIP):0`に向かって接続をすれば良い。

### VNCサーバーの自動起動

[参考サイト](https://qiita.com/iwatake2222/items/a3bd8d0527dec431ef0f#%E6%96%B9%E6%B3%951-tigervnc%E3%82%92%E4%BD%BF%E3%81%86)よりほぼコピペ。

自動起動の設定。

~~~shell
sudo vi /etc/systemd/system/x0vncserver.service
~~~

以下を記入。

~~~
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target
After=network.target remote-fs.target nss-lookup.target
After=x11-common.service 

[Service]
Type=forking
User=(自分のユーザー名)
Group=(自分のユーザーグループ名)
WorkingDirectory=/home/(自分のユーザー名)
ExecStart=/bin/sh -c 'sleep 10 && /usr/bin/x0vncserver -display :0  -rfbport 5900 -passwordfile /home/(自分のユーザー名)/.vnc/passwd &'

[Install]
WantedBy=multi-user.target
~~~

登録。

~~~shell
$ sudo systemctl enable x0vncserver
~~~

### 解像度の設定

また[参考サイト](https://qiita.com/iwatake2222/items/a3bd8d0527dec431ef0f#%E6%96%B9%E6%B3%951-tigervnc%E3%82%92%E4%BD%BF%E3%81%86)よりほぼコピペ。

~~~shell
$ sudo vi /etc/X11/xorg.conf
~~~

以下を末尾に追記。

~~~
Section "Monitor"
   Identifier "DSI-0"
   Option    "Ignore"
EndSection

Section "Screen"
   Identifier    "Default Screen"
   Monitor        "Configured Monitor"
   Device        "Default Device"
   SubSection "Display"
       Depth    24
       Virtual 1280 800
   EndSubSection
EndSection
~~~


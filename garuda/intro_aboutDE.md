# デスクトップ環境について

WindowsやMacでは起動後にデスクトップが立ち上がるのが普通だが、Linuxの世界ではそれは普通ではない。

## 概要

基本的にLinuxを立ち上げると画面に映るのは黒い画面に文字ばかり打つターミナル（またはコンソール、黒い画面とか呼ばれるアレ）である。

そこから`startx`とか打つと設定に応じてデスクトップ環境が立ち上がる。もちろんデスクトップ環境がインストールされていなければ立ち上がらない。

## デスクトップ環境とツールキット

デスクトップ環境というのはLinux上で動く一種のソフトウェアという認識なので、同じLinux上でもそのソフトウェアを入れ替えればデスクトップ環境を入れ替えることができる。これはWindowsやMacにはない考え方。

ユーザー側で気を付けておかないといけないのは、「導入したいソフトが自分のデスクトップ環境で動くかどうか確認するためには、ツールキットに注目しないといけない」ということ。

ツールキットというのは開発者向けのライブラリのことで、ユーザーには直接関係はない。ただし、デスクトップ環境もソフトウェアの一種なので「デスクトップ環境で何かのソフトを動かす＝あるソフトウェア上で別のソフトウェアを動かす」ということになり、「あるソフトウェア」と「別のソフトウェア」は同じツールキットを使わないといけない。

後でいくつか紹介するが、調べてみるとどうも主要なツールキットはGTKとQtがあるらしい。GTKを使って作られたデスクトップ環境では、GTKを使って作られたソフトしか動かせない。Qtも同じ

実際はいろいろあるので「Linux デスクトップ環境」とかで調べてもらうとして、代表的なものをいくつか見てみる。

### GNOME

Ubuntu Desktopを導入して立ち上げると動くのがGNOMEというデスクトップ環境。ツールキットはGTK。

### Xfce

軽量なデスクトップ環境。ツールキットはGTK。

### MATE

GNOMEから派生したデスクトップ環境。ツールキットはGTK。

### KDE

今回のGaruda KDE Dragonized EditionはこのKDEというデスクトップ環境が動く。KDE Plasmaとも呼ばれたりするけど、詳しくはよく分からん。Plasmaと付けば最近のものなんだな、ぐらいの認識。ツールキットはQt。

### LXDE

Lightweight X11 Desktop Environmentの略で、軽量なデスクトップ環境。ツールキットはGTK。後にツールキットをQtに変えたLXQtも作られ、現在ではLXDEの開発は中止されLXQtの開発が続いている。

### Raspberry Piでは？

LXDEをベースに作ったPIXELというデスクトップ環境が動いているらしい。ということはツールキットはGTKか。

## デスクトップ環境のベースシステム

実は上記のデスクトップ環境というのはベースである「ウィンドウシステム」上で動いている。ウィンドウシステムが無いとデスクトップ環境も動かない。基本的にユーザーが気にするものでは無い。

### X（またはX11）

正式名称はX Window System。LinuxでXと言えばほぼコレと言えるぐらいにスタンダードらしい。基本的にKDEはX Window System上で動く（とWikipediaが言ってた）。

`startx`コマンドで最終的に動くのはコイツ。

X11を使うとネットワーク越しにGUIソフトを実行することができる。

### WayLand

後発のウィンドウシステム。X11と比べるとネットワーク越しのGUIソフト実行はできないけど、その分速さを追求しているとのこと。

## 参考

[デスクトップ環境の評価 \- 一般的なデスクトップ](https://freebsd.sing.ne.jp/desktop/ev/02.html)
[OSS開発ツール/GUIツールキット \- Wikibooks](https://ja.wikibooks.org/wiki/OSS%E9%96%8B%E7%99%BA%E3%83%84%E3%83%BC%E3%83%AB/GUI%E3%83%84%E3%83%BC%E3%83%AB%E3%82%AD%E3%83%83%E3%83%88)
[Introducing PIXEL \- Raspberry Pi](https://www.raspberrypi.org/blog/introducing-pixel/)
[デスクトップ環境 \- ArchWiki](https://wiki.archlinux.jp/index.php/%E3%83%87%E3%82%B9%E3%82%AF%E3%83%88%E3%83%83%E3%83%97%E7%92%B0%E5%A2%83)
[GTK \+とQTの違いは何ですか？ \| \- テクノロジ \- 2021](https://ja.texasucanpaint.com/difference-gtk-qt-9264)

[startx 〜 デスクトップ環境の起動まで \| Netsphere Laboratories](https://www.nslabs.jp/startx.rhtml)
[startxコマンドの使い方: UNIX/Linuxの部屋](http://x68000.q-e-d.net/~68user/unix/pickup?startx)
[Linux基礎\(X Window System\) \- Qiita](https://qiita.com/kakkie/items/c6ccce13ce0beaefaad1)
[【サルでも分かる】X11入門 \- \(O\+P\)ut](https://www.mtioutput.com/entry/2018/11/08/090000)
[Linuxでよく聞く「Xとは？」とX11のインストール方法](https://eng-entrance.com/linux-gui-x11)
[Wayland \| Linux技術者認定 LinuC \| LPI\-Japan](https://linuc.org/study/knowledge/496/)


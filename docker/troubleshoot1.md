# SSHでX11フォワーディングができないとき

参考：[MacでUbuntuにx接続しようとしたらトラブった ｜ enjoyall](https://enjoyall.comichi.com/mac_x_connection/)

1. `ssh -Y`で接続する。
2. 接続先のホームディレクトリにある`.Xauthority`の所有者が`root`になっていたら接続ユーザーに変更する。

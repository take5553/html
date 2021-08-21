# GUIの有効無効のスイッチ

よく使うくせによく忘れるからメモ。

## その場でスイッチ

~~~shell
$ sudo init 3 # stop the desktop
$ sudo init 5 # restart the desktop
~~~

## 再起動しても継続

~~~shell
$ sudo systemctl set-default multi-user.target # disable desktop on boot
$ sudo systemctl set-default graphical.target # enable desktop on boot
~~~


# OS再起動後にGitLabが立ち上がらない問題

インストール直後はちゃんと立ち上がったのにOS再起動後にGitLabにアクセスできなくなる問題。

## 環境

RHEL 8

## 調査

GitLabの状態を調べると`runsv`というものが動いていない。

~~~shell
$ sudo gitlab-ctl status
fail: alertmanager: runsv not running
fail: gitaly: runsv not running
fail: gitlab-exporter: runsv not running
fail: gitlab-workhorse: runsv not running
fail: grafana: runsv not running
fail: logrotate: runsv not running
fail: nginx: runsv not running
fail: node-exporter: runsv not running
fail: postgres-exporter: runsv not running
fail: postgresql: runsv not running
fail: prometheus: runsv not running
fail: puma: runsv not running
fail: redis: runsv not running
fail: redis-exporter: runsv not running
fail: sidekiq: runsv not running
~~~

これは`gitlab-runsvdir`というサービスのことらしい。その状態を調べてみる。

~~~shell
$ sudo systemctl status gitlab-runsvdir
● gitlab-runsvdir.service - GitLab Runit supervision process
Loaded: loaded (/usr/lib/systemd/system/gitlab-runsvdir.service; enabled; vendor preset: disabled)
Active: inactive (dead)
~~~

動いていない。これは`systemd`が`multi-user.target`を処理している間に立ち上がるものらしい。

状況確認。

~~~shell
$ sudo systemctl -t target
UNIT                   LOAD   ACTIVE   SUB    JOB   DESCRIPTION                
basic.target           loaded active   active       Basic System               
cryptsetup.target      loaded active   active       Local Encrypted Volumes    
getty.target           loaded active   active       Login Prompts              
graphical.target       loaded inactive dead   start Graphical Interface        
local-fs-pre.target    loaded active   active       Local File Systems (Pre)   
local-fs.target        loaded active   active       Local File Systems         
multi-user.target      loaded inactive dead   start Multi-User System          
network-online.target  loaded active   active       Network is Online          
network-pre.target     loaded active   active       Network (Pre)              
network.target         loaded active   active       Network                    
nfs-client.target      loaded active   active       NFS client services        
nss-user-lookup.target loaded active   active       User and Group Name Lookups
paths.target           loaded active   active       Paths                      
remote-fs-pre.target   loaded active   active       Remote File Systems (Pre)  
remote-fs.target       loaded active   active       Remote File Systems        
rpc_pipefs.target      loaded active   active       rpc_pipefs.target          
rpcbind.target         loaded active   active       RPC Port Mapper            
slices.target          loaded active   active       Slices                     
sockets.target         loaded active   active       Sockets                    
sound.target           loaded active   active       Sound Card                 
sshd-keygen.target     loaded active   active       sshd-keygen.target         
swap.target            loaded active   active       Swap                       
sysinit.target         loaded active   active       System Initialization      
timers.target          loaded active   active       Timers                     

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
JOB    = Pending job for the unit.
~~~

`multi-user.target`と`graphical.target`が動いていない。`start`JOBが保留状態にあるらしい。

何で詰まっているのか確認。

~~~shell
$ sudo systemctl list-jobs
JOB UNIT                                 TYPE  STATE  
122 multi-user.target                    start waiting
291 systemd-update-utmp-runlevel.service start waiting
289 gitlab-runsvdir.service              start waiting
121 graphical.target                     start waiting
252 plymouth-quit-wait.service           start running
~~~

`plymouth-quit-wait`が原因らしい。

## 解決方法

GUI上でログインしたら解決した。`plymouth-quit-wait`はログイン待ちのためにあるのか？

いらないならアンインストールしてしまえ、という方法もあるらしい。→[Fedora 25: plymouth-quit-waitが遅い問題 - Narrow Escape](https://www.hiroom2.com/2016/12/21/fedora-25-plymouth-quit-wait%E3%81%8C%E9%81%85%E3%81%84%E5%95%8F%E9%A1%8C/)

おそらくこの問題はインストール時に「サーバー（GUI）」を選択したからであって、GUIでのログインが完了して初めて`systemd`の起動プロセス全てが完了すると推測される。

## 余談

起動時間を調べるのは以下。

~~~shell
$ systemd-analyze
Startup finished in 1.245s (kernel) + 2.601s (initrd) + 12min 39.990s (userspace) = 12min 43.837s
~~~

詳細に調べるなら以下。

~~~shell
$ systemd-analyze blame
12min 34.239s plymouth-quit-wait.service
4.237s dnf-makecache.service
2.853s NetworkManager-wait-online.service
1.938s rhsm.service
1.701s kdump.service
1.495s dracut-initqueue.service
(以下略)
~~~

`plymouth-quit-wait`に依存するサービスの調べ方。

~~~shell
$ sudo systemctl list-dependencies --reverse plymouth-quit-wait
plymouth-quit-wait.service
● └─multi-user.target
●   └─graphical.target
~~~

## 解説

### `systemd`

システムの起動・終了やデーモンの管理機能を提供するもの。`systemctl`で制御しているのはこの`systemd`。

Linuxのカーネルが読み込まれた後に実行される。

処理単位として`Unit`というものがあり、複数の`Unit`をまとめたものを`target`と呼ぶ。雑な理解としては`target` = `runlevel`。GUI無しのサーバーの場合普通は`multi-user.target (runlevel = 3)`、GUI有りのサーバーの場合は`graphical.target (runlevel = 5)`。

詳しくは参考サイトへ。

## 参考

`systemd`

[「Systemd」を理解する ーシステム起動編ー | ギークを目指して](http://equj65.net/tech/systemd-boot/)
[「Systemd」を理解する ーシステム管理編ー | ギークを目指して](http://equj65.net/tech/systemd-manage/)
[systemctl コマンド - Qiita](https://qiita.com/sinsengumi/items/24d726ec6c761fc75cc9)
[systemd & systemctl - とみぞーノート](https://wiki.bit-hive.com/tomizoo/pg/systemd%20%26%20systemctl)

GitLab

[Common installation problems | GitLab](https://docs.gitlab.com/omnibus/common_installation_problems/#gitlab-runsvdir-not-starting)

`plymouth`

[boot — plymouth-quit-wait.service + ubuntu 18.04による起動の問題](https://www.it-swarm-ja.com/ja/boot/plymouthquitwaitservice-ubuntu-1804%E3%81%AB%E3%82%88%E3%82%8B%E8%B5%B7%E5%8B%95%E3%81%AE%E5%95%8F%E9%A1%8C/998391215/)
[Plymouth その1 - Plymouthとは・Plymouthのテーマとは・Plymouthのテーマを切り替えるには - kledgeb](https://kledgeb.blogspot.com/2016/01/plymouth-1-plymouthplymouthplymouth.html)
[10.3. Plymouth Red Hat Enterprise Linux 7 | Red Hat Customer Portal](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/desktop_migration_and_administration_guide/plymouth)


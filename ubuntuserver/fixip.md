# Ubuntu Serverでの固定IP変更方法

Ubuntuはインストール時にIPをどうするか聞かれるので初回は特に困らないけど、変更するときにRaspberry Piとは同じ方法ではないので記しておく。

## 環境

Ubuntu Server 20.04 LTS 

## 手順

 Ubuntu Serverにログインして以下を打つ。

~~~shell
$ sudo nano /etc/netplan/00-installer-config.yaml
~~~

そうすると以下のようなものが表示される。

~~~
# This is the network config written by 'subiquity'
network:
    ethernets:
        enp1s0:
            addresses:
            - 192.168.1.203/24
            gateway4: 192.168.1.1
            nameservers:
                addresses:
                - 192.168.2.1
                search:
                - 192.168.2.1
                - 8.8.8.8
    version: 2
~~~

適当に必要なところを変えて保存終了。

その後に

~~~shell
$ sudo netplan apply
~~~

で設定を反映させる。

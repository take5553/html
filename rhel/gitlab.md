# GitLabをRHELにインストールしてGitのリポジトリサーバーにする

訳あってUbuntu ServerにインストールしたGitLabをRHELにもインストールする。

## 環境

RHEL 8

## 手順

### 準備

~~~shell
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=httpd
$ sudo systemctl reload firewalld
$ sudo dnf install postfix
$ sudo systemctl enable postfix
$ sudo systemctl start postfix
~~~

### GitLabパッケージのリポジトリへの追加とインストール

~~~shell
$ crul https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
$ sudo dnf install gitlab-ce
~~~

## GitLabの初期設定

~~~shell
$ sudo nano /etc/gitlab/gitlab.rb
~~~

以下を書き換え。

~~~
external_url 'http://takeshi-rhel-gitlab'
gitlab_rails['time_zone'] = 'Asia/Tokyo'
~~~

設定を適用。

~~~shell
$ sudo gitlab-ctl reconfigure
~~~

## PCからアクセス

名前解決を追加。

### Linuxの場合

以下を開く。

```
$ sudo nano /etc/hosts
```

末尾に以下を記入。IPアドレスはサーバーのローカルIP。

```
192.168.1.204 takeshi-rhel-gitlab
```

### Windowsの場合

メモ帳を管理者権限で開いて、メニューの「開く」で`c:\Windows\System32\drivers\etc\hosts`を指定してファイルを開く。末尾に以下を追記。

```
192.168.1.204 takeshi-rhel-gitlab
```

### ブラウザでアクセス

```
http://takeshi-rhel-gitlab
```

## まとめ

後は普通のGitLabの使い方と一緒。

## 参考

[GitLabをダウンロードしてインストール | GitLab.JP](https://www.gitlab.jp/install/#centos-8)
[CentOS7にGitLabインストール - Qiita](https://qiita.com/hiren/items/dbf7ee9b2fe651d1298f)


# ドメインを無料で取得する（No-IP使用）

前回の記事で外部ネットワークから自宅のグローバルIPを使ってRaspberry Piにアクセスする方法を紹介したが・・・

グローバルIPは実はプロバイダーの都合で変わる。

グローバルIPは1家に1つ（厳密には1契約に1つ？）であることは変わらないけど、IP自体は固定ではないことがある。（プロバイダーと「固定グローバルIP」みたいな契約をしていたら別）

なので、自宅のグローバルIPに紐づくドメイン（URLのようなもの）を取得して、いつもそのドメインを打てば自宅のRaspberry Piに繋がるという状態を作っていく。

ちなみにここで取得したドメインは以後Webサーバーの公開にも使うのでそれを考えて決めること。

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 1.4

## 概要

※No-IPは30日ごとにログインまたは生存確認的なものをNo-IPに送らないといけない。それが面倒な人は別のDDNSサービスを探してください。



- 無料DDNSサービス「No-IP」
  [Free Dynamic DNS - Managed DNS - Managed Email - Domain Registration - No-IP](https://www.noip.com/)
  
- No-IPに登録してドメインをゲットする（下記サイトのStep3まで）
  
  [Free Dynamic DNS : Getting Started Guide](https://www.noip.com/support/knowledgebase/getting-started-with-no-ip-com/)
  
- 「No-IP」が提供するDUC（Dynamic Update Client）をRaspberryPiにインストールする。
  [How to Install the No-IP DUC on Raspberry Pi | Support | No-IP Knowledge Base](https://www.noip.com/support/knowledgebase/install-ip-duc-onto-raspberry-pi/)

## 前提知識

### ドメインとは

専門用語を並べるつもりはないのでざっくり言うと、ドメインとはIPの別名。URLのようなもの。「google.com」はGoogleのドメイン。

参考：
[IPアドレスでWebサイトにアクセス？ \| IT情報メディア「LIVRA」](https://livra.geolocation.co.jp/iplearning/691/)
[ドメイン名・ホスト名・FQDNって？ \| IT情報メディア「LIVRA」](https://livra.geolocation.co.jp/iplearning/250/)

#### 実験

ブラウザのアドレスバーに「172.217.161.196」と打ってみよう。ドメインはIPの別名だという感覚が分かるはず。

### DDNSとは

ドドンス。Dynamic Domain Name System。詳しくは調べてもらうとして、かなり主観的に説明すると「無料でドメインがもらえるサービス。定期的にIPを知らせてドメインとの紐づけを更新しないともらったドメインが消滅する」みたいな感じ。定期的にIPを通知するツールが提供されていたり、フリーで公開されていたりする。

## 方法

[Free Dynamic DNS \- Managed DNS \- Managed Email \- Domain Registration \- No\-IP](https://www.noip.com/)

ウチのサイトのドメインもここで取得してる。他のサービスを探してみたい人は[無料ダイナミックDNS比較](https://www.kooss.com/ddns/)をチェック。

自分はもう登録しちゃったので、スクリーンショットで手順を紹介することができない。その代わり、公式の導入ガイドである以下のページを、ポイントだけ訳しておく。

### No-IPでドメインを取得する

[Free Dynamic DNS : Getting Started Guide](https://www.noip.com/support/knowledgebase/getting-started-with-no-ip-com/)

* Step1: アカウントを作る

  [Create a Free Dynamic DNS No\-IP Account](https://www.noip.com/sign-up)にアクセスしてアカウントを作る。「Hostname」に希望のURLを入力する。自分が登録したときはHostnameに「arcticstreet」と入力し、「.ddns.net」を選択した。有料プランもあるから、「Free Sign Up」を選択するようにしよう。

* Step2: アカウント確認

  メールが飛んできて、中にあるリンクを踏めばアカウント確認完了。

* Step3: ログインしよう

  E-mailアドレスかusernameをIDとして入力し、決めたパスワードをいれて「Log In」をクリックすればログインできるはず。

* Step4: ドメインをアカウントに登録する

  Step1でドメイン決めてるはずだけど、もしもう一つ追加したくなったら（無料プランなら最大3つまでドメインがもらえる）これこれこうしてねって書いてある。

  ログイン後のダッシュボードで画面左側のメニューから「My Services」→「DNS Records」に進んでStep1で決めたドメインが表示されていたら不要。

  一応もう一つ作りたい場合は同じ画面内に表示されている「Create Hostname」をクリックして、おそらく「Hostname」「Hostname Type→DNS Hostname(A)」「IP Address」だけ決めて他は無視して一番下までスクロールして「Add Hostname」をクリックしたら大丈夫。

ここまででとりあえずドメインと自宅のグローバルIPが結び付いている。プロバイダーと固定IPアドレス契約を結んでいるならここで目的完了。でもほとんどの場合はそうじゃないので、以下でグローバルIPが変わっても再紐づけを自動でやってくれるツールを導入する。

### IPの自動更新ツール「Dynamic Update Client（DUC）」をRaspberry Piにインストールする

ここからは以下に従う。

[How to Install the No\-IP DUC on Raspberry Pi \| Support \| No\-IP Knowledge Base](https://www.noip.com/support/knowledgebase/install-ip-duc-onto-raspberry-pi/)

以下、Raspberry Piにログインして作業。

ホームディレクトリ配下に「noip」というディレクトリを作りそこに移動する。

~~~shell
$ mkdir ~/noip
$ cd ~/noip
~~~

圧縮ファイルを取得して展開する。

~~~shell
$ wget https://www.noip.com/client/linux/noip-duc-linux.tar.gz
$ tar vzxf noip-duc-linux.tar.gz
~~~

「noip-2.1.9-1」というディレクトリが直下にできるのでそこに移動。

~~~shell
$ cd noip-2.1.9-1
~~~

インストールする。

~~~shell
$ sudo make
$ sudo make install
~~~

上の二つのコマンドを打つとインストーラーが起動して、No-IPアカウントのusernameとパスワードを使ってログインしろとか他の色々を聞いてくる。特にどれぐらいの頻度で（How often）IPを更新するかを聞いてきた時は`5`以上の数字を入力しなければならない。これは5分に1度の間隔でドメインとIPの紐づけの更新を行うということらしい。`30`とかでも可。

ツールを実行。

~~~shell
$ sudo /usr/local/bin/noip2
~~~

ツールが正しく動いているか知りたかったら以下のコマンドを打つ。

~~~shell
$ sudo noip2 -S
~~~

この`-S`は大文字。

### 本当にこれでドメインをゲットできたの？

確認するためには**外部ネットワーク上で**PowerShellから以下を打つ。

~~~shell
> ssh (ユーザー名)@(取得したドメイン) -p (ルーターのポートフォワーディングで開放したポート番号)
~~~

家の中でやってたらおそらくは繋がらない。

どうしても家の中でやりたい！という人は[前回](portforwarding.html)の最後で紹介したスマホのやり方で、AddressをIPから今取得したドメインに変えてつなげてみる。
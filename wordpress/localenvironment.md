# ローカル上にテスト環境を整える

本番環境でテストするのはよろしくないので、ローカル上にもWordPressをインストールして確認してからRaspberry Piにアップするようにする。

## 環境

- ローカル（PC側）
  - Windows10

## 概要

[LOCAL](https://localwp.com/)を使ってローカルにWordPressをインストールする。

## 方法

[LOCAL](https://localwp.com/)にアクセスして「OR DOWNLOAD FOR FREE」をクリック。

![image-20201027213028680](image/localenvironment/rs-image-20201027213028680.png)

プラットフォームでWindowsを選択。

![image-20201027213452060](image/localenvironment/rs-image-20201027213452060.png)

名前とメールアドレスと電話番号を入れろと出てくる。電話番号まで入れるのは怪しすぎるので空欄にしたらダウンロードが始まったからいらんっぽい。

![image-20201027213523056](image/localenvironment/rs-image-20201027213523056.png)

インストーラーが立ち上がって、適当にポチポチしたらインストール完了。

LOCALを立ち上げると、規約同意のあとエラーリポートをオンにするか聞かれる。お好きなように。

![image-20201027213823668](image/localenvironment/rs-image-20201027213823668.png)

よく分からないアプリの宣伝をされるので、「×」をクリック。

![image-20201027213948169](image/localenvironment/rs-image-20201027213948169.png)

「まだサイトを作ってないようだね！」と表示されるので、「CREATE A NEW SITE」をクリック。

![image-20201027214123519](image/localenvironment/rs-image-20201027214123519.png)

サイト名を聞かれる。ローカルテスト環境なので適当につける。

![image-20201027214231505](image/localenvironment/rs-image-20201027214231505.png)

環境を選べと聞かれる。Customを選び、できるだけリモート環境に近いバージョンを構築する。

* PHP 7.3.5
* Nginx 1.16.0
* MariaDB 10.4.10

を選択。

![image-20201027214434566](image/localenvironment/rs-image-20201027214434566.png)

ローカルWordPress用のユーザー名とパスワードとメアドを聞かれる。メアドって入力する必要ある？とりあえずデフォルトで何か入っていたので、そのままにする。

![image-20201027214818639](image/localenvironment/rs-image-20201027214818639.png)

インストール中にWindowsがなんか色々聞いてくるけど、適当に許可。

![image-20201027215034853](image/localenvironment/rs-image-20201027215034853.png)

すぐインストールが終わった。VIEW SITEをクリックするとブラウザが立ち上がって今作ったWordPressが見れるらしい。

![image-20201027215204056](image/localenvironment/rs-image-20201027215204056.png)

英語かよ。

![image-20201027215236298](image/localenvironment/rs-image-20201027215236298.png)

先の画面でADMINをクリックするとブラウザ上にログイン画面が表示される。

![image-20201027215349777](image/localenvironment/rs-image-20201027215349777.png)

先のメニューでタイトルの下に小さく表示されているパスがどうもWordPressのインストール場所らしい。「＞」マークをクリック。

![image-20201027215613543](image/localenvironment/rs-image-20201027215613543.png)

む、なんかちゃう。

![image-20201027215843370](image/localenvironment/image-20201027215843370.png)

`app`→`public`と進めばWordPressのファイル群があった。この`wp-content`→`themes`の中に自作テーマを入れるとローカル環境でも管理画面で自作テーマが選べるはず。

![image-20201027220034760](image/localenvironment/image-20201027220034760.png)

できた。相変わらずひどい。

![image-20201027220924420](image/localenvironment/rs-image-20201027220924420.png)

とりあえずこれでローカル上で確認してからアップするということができるようになった。
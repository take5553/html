# 準備2（GitHubにリモートリポジトリ作成、XAMPPインストール）

## 環境

* ローカル
  * Windows 10

## GitHubにリモートリポジトリを作成

自分はもうすでにGitHubのアカウントを持っているので、まだの人は適当にググってアカウントを作ろう。

頑張って「新しいリポジトリを作成する」ところまでたどり着く。

![image-20201206154343247](image/preparation2/rs-image-20201206154343247.png)

Repository nameを`php-bbs`とし、後は適当にそのままにして作成。

その後表示されるページの中段部分で、「SSH」をクリックし、その右に表示されている文字列をクリップボードにコピー。

![image-20201206154656449](image/preparation2/rs-image-20201206154656449.png)

VSCode上のGitペインでリモートの追加をする。

![image-20201206155034940](image/preparation2/image-20201206155034940.png)

画面上部に何やら出てきた。どうも「GitHubからリモートを追加する」でいけるっぽい。

![image-20201206155125702](image/preparation2/rs-image-20201206155125702.png)

![image-20201206155215716](image/preparation2/rs-image-20201206155215716.png)

VSCode上でGitHubにサインインするのか。

![image-20201206155358317](image/preparation2/image-20201206155358317.png)

許可して何回かGitHubからの問い合わせにもポチポチしたら、自分のアカウントに紐づいているリポジトリが表示されるようになった。`php-bbs`を選択。

![image-20201206155534685](image/preparation2/image-20201206155534685.png)

リモート名は`origin`で。おそらくこれで大丈夫なはず。

![image-20201206155628012](image/preparation2/image-20201206155628012.png)

プッシュしてみる。

![image-20201206155728575](image/preparation2/image-20201206155728575.png)

よく分からんけどOK。

![image-20201206155754371](image/preparation2/image-20201206155754371.png)

GitHubへのログインを求められる。

![image-20201206155819394](image/preparation2/image-20201206155819394.png)

ログインしてしばらくしたらGit Graphに`origin`も表示されるようになった。

![image-20201206155933987](image/preparation2/image-20201206155933987.png)

ちゃんとアップされている。

![image-20201206160010059](image/preparation2/rs-image-20201206160010059.png)

これでVSCode上でGitHubへのプッシュもできるようになったのか。便利。

## XAMPPインストール

[XAMPP Installers and Downloads for Apache Friends](https://www.apachefriends.org/jp/index.html)

![image-20201206160627120](image/preparation2/rs-image-20201206160627120.png)

Windows向けをDL。PHP8.0.0ってなってるけど、Raspberry Piに入っているPHPは7.3だったかな？まあいいか。不具合が出たら立ち向かっていこう。

※[不具合が出たのでXAMPP 7.4.13をオススメする。](troubleshooting1.html)

インストーラーをダウンロードして起動したら「ウィルスソフト動いてるみたいだけど、インストール遅くなるかも。」と言われた。ウィルスソフトを一時停止して再度やってみる。

![image-20201206160944216](image/preparation2/image-20201206160944216.png)

今度は「ユーザーアカウントコントロール（UAC）が動いてる！`C:\Program Files`へのインストールを避けるか、このセットアップ後にUACを切ってね」って言われた。インストールフォルダを別のところにしたらいいのか。

![image-20201206161333240](image/preparation2/image-20201206161333240.png)

インストール、始まるよー。

![image-20201206161348787](image/preparation2/image-20201206161348787.png)

とりあえず必要なのは「Apache、MySQL、PHP」だけど全部入れよう。で、必要があったら使っていくことにしよう。

![image-20201206161418463](image/preparation2/image-20201206161418463.png)

インストールフォルダ。なんや、このままでええやん。

![image-20201206161542721](image/preparation2/image-20201206161542721.png)

言語は英語かドイツ語か。その2択なら英語しかない。

![image-20201206161609952](image/preparation2/image-20201206161609952.png)

「Bitnamiをよろしく！」

![image-20201206161721487](image/preparation2/image-20201206161721487.png)

いきまーす。

※途中で勝手に再起動するみたい。未保存のものは保存しておこう。

![image-20201206161745187](image/preparation2/image-20201206161745187.png)

インストールが終わって、XAMPPのコントロールパネルを立ち上げる。

![image-20201206162622505](image/preparation2/image-20201206162622505.png)

このままApacheを立ち上げてもいいけど、Apacheのドキュメントルートを今自分が作ってるドキュメントルートに設定しておこう。

Apacheの右にある「Config」をクリック。メモ帳が立ち上がるので、`DocumentRoot`という項目を探す。`DocumentRoot`とその下の`<Directory>`の中に書かれているパスを自分のドキュメントルートに書き換える。

自分の場合は`D:\work\HTML\raspberrypi-server\test\html`。

![image-20201206163430679](image/preparation2/image-20201206163430679.png)

然らばスタート。

![image-20201206163815840](image/preparation2/image-20201206163815840.png)

ブラウザを立ち上げ、アドレスバーに`localhost/bbs`と打ち込めば「Hello PHP！」と表示されるはず。

![image-20201206163957493](image/preparation2/image-20201206163957493.png)

これで、テスト環境は整った。こんなんだったら、[WordPressのローカル環境を整えるとき](../wordpress/localenvironment.html)にLocalを使わずにXAMPPを使っておけばよかったね。

※後日追記：`bbs/`を付けるのが面倒だったからapacheのドキュメントルートの場所をワークフォルダ（自分の場合は`D:\work\HTML\raspberrypi-server\test\html\bbs`）に設定した。今後はブラウザから`localhost`にアクセスすれば、ワークフォルダのファイルが表示される、ということにする。

## VSCodeにPHP実行ファイルの場所を教える

[前回](preparation.html)VSCodeから「PHP実行ファイルの場所を教えてくれ」と言われていたので教える。

VSCodeを開いて、`index.php`を開いたらまだ警告を出してくれているので、それに乗っかる。

![image-20201206164737516](image/preparation2/image-20201206164737516.png)

「設定を開く」ボタンをクリックすると、「setting.jsonで編集」とあるので、クリック。

![image-20201206164831469](image/preparation2/image-20201206164831469.png)

ここに書けばいいのね。

![image-20201206164853813](image/preparation2/image-20201206164853813.png)

PHPの実行ファイルの場所は`C:\xampp\php\php.exe`なんだけど、`.json`ファイル上では`\`はエスケープ文字扱いになるので、書くときは`C:\\xampp\\php\\php.exe`とする。

![image-20201206165059250](image/preparation2/image-20201206165059250.png)

保存して終了。これで`.php`を開いたときに文法をチェックしてくれるようになった。多分。
# Docker入門編まとめ

1. 「DockerでUbuntuコンテナ立ててその中で普通にWebサーバー立ててPHP動くようにしたら、シンプルな仮想化っぽくて分かりやすいんじゃね？」
   * 1コンテナ1プロセスという原則を知る
2. 「ほんならPHPを別コンテナにしたらええってこと？」
   * Ubuntuコンテナに毎回Nginxを入れるのが面倒になる
3. 「Nginxの公式イメージが配布されてるからそれ使えってこと？」
   * 毎度複数のコンテナを立ち上げるコマンドが面倒になる。
4. 「docker-composeを使えば複数コンテナ一発で立ち上がるん？」
   * 公式イメージに対して都合のいい設定を加えないといけない
5. 「公式イメージに設定を注入したイメージを自分で作れってこと？」
   * 今ここ

~~~
my-docker
└── sample-app
    ├── docker-compose.yml
    ├── settings
    │   ├── nginx
    │   │   ├── default.conf
    │   │   └── Dockerfile
    │   └── phpfpm
    │       ├── Dockerfile
    │       └── php.ini
    └── src
        └── index.php
~~~

もしこれをGit管理下に置いて、GitHubに上げて、誰かにDLしてもらうとしたら、その人にやってもらうことは

1. `docker-compose up -d`（`start.sh`みたいなシェルスクリプトにしてもいいかもしれない）
2. メインPCのブラウザにJetson NanoのIPを打ち込んで、Webページが表示されることを確認
3. 終わったら`docker-compose down`（`end.sh`とかでもOK）

これがもしDockerを使わなかったら、

1. Nginxをインストールしてもらう
2. Nginxの設定をこちらで用意した設定ファイルで上書きしてもらう
3. PHP-FPMをインストールしてもらう
4. PHPの設定も上書きしてもらう
5. `index.php`をドキュメントルートに入れてもらう
6. NginxとPHP-FPMの立ち上げ＆確認
7. Webページの表示確認

となる。そういうシェルスクリプトを組めば良いと言えばそれまでかもしれないけど、まあ面倒だし、エラーが出て上手くいかないときに、その人が詳しい人じゃなかったら悪夢でしかない。

Dockerなら動作確認までできるから「ほぼ動くんじゃね？」ぐらいで安心できる。

そういうことか。

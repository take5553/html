# コンテナ内の音声ファイルを再生する

参考リンク：[Passing audio into docker container | NTTData4 - lab virtual assistant v2 Development Blog](https://comp0016-team-24.github.io/dev/problem-solving/2020/10/30/passing-audio-into-docker.html)

なぜ以下の手順でいけるのかは、上のリンク先を熟読すべし。

## 前提

コンテナ内で`pulseaudio`がインストールされている。

## `docker`コマンドでコンテナを立ち上げる場合

~~~
docker run \
  -v /run/user/$UID/pulse/native:/run/user/$UID/pulse/native \
  -e PULSE_SERVER=unix:/run/user/$UID/pulse/native \
  -u $UID:$GID \
  ...
~~~

試してないけど動くと思う。多分。

## `docker-compose.yml`で立ち上げる場合

`docker-compose.yml`

~~~yaml
services:
    main:
        environment:
          - PULSE_SERVER=unix:/run/user/$UID/pulse/native
        volumes:
          - /run/user/$UID/pulse/native:/run/user/$UID/pulse/native
		user: "${UID}:${GID}"
		...
~~~

そして以下を打つ。

~~~shell
$ export UID
$ export GID
$ docker-compose up
~~~

コンテナ内に`root`ユーザーしかいない場合は`user`の行は削除できる。`root`だから別にユーザーID関係ないし。

## その後

これでホストとコンテナの音声デバイスが繋がったことになるから、後はコンテナ内でどうにかして音を鳴らせばホスト側のスピーカーから音が鳴るはず。

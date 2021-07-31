# Docker編

## 環境

これは[Jetson AI Certification編](../jetsoncert/)の一部として書くので、環境は基本的に以下とする。

* Jetson Nano 2GB
* JetPack 4.5.1
* メインPCからJetson NanoへSSHでリモート接続している

ただし、JetPackは基本的にUbuntuをベースにしているので、Linux上であれば同じことができるかもしれない。GPUを使うような場面では環境依存アリ。

ちなみにDockerはすでにJetPackにインストールされているので、Dockerのインストール作業は省略。

## 参考

[Orientation and setup | Docker Documentation](https://docs.docker.com/get-started/)

## 目標

Deep Leaning技術を使用したアプリを作成し、Dockerイメージと一緒に配布する。

## 目次

### 入門編

Jetson Nano上で進めるけど、特にJetson Nanoでないといけないわけではない部分。

* [とにかくコンテナを動かす](getting_started.html)
* [Dockerイメージの作り方](create_an_image.html)
* [イメージファイルの共有とアーキテクチャーの罠](share_the_image.html)
* [Ubuntuコンテナでマウント](mount_in_ubuntu.html)
* [複数コンテナ](multi_containers.html)
* [Nginx専用コンテナを使ってもう一度](specific_container.html)
* [複数のコンテナを一発で立ち上げる](docker_compose.html)
* [イメージファイルを作りながら複数コンテナを一発で立ち上げる](docker_compose_with_dockerfile.html)
* [まとめ](matome1.html)

### Jetson Nano編

Jetson Nanoにバリバリ依存した内容。

* 公式イメージのDL


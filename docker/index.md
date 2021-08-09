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

* [公式イメージ](jetson_containers.html)
* [NVIDIA L4T Baseイメージ1　コンテナ内でGUIアプリを立ち上げ、ホスト側に表示](l4t_base1.html)
* [NVIDIA L4T Baseイメージ2　このイメージに入っているもの確認](l4t_base2.html)
* [NVIDIA L4T PyTorchイメージ](pytorch_container.html)
* [コンテナ上でGUIアプリ開発　環境構築1](gui_development.html)
* [コンテナ上でGUIアプリ開発　アプリ試作](opencv_and_tkinter.html)
* [コンテナ上でGUIアプリ開発　環境構築2](gui_development2.html)
* コンテナ上でGUIアプリ開発　環境構築まとめ

## 関連リンク

### アーキテクチャーの壁を乗り越える

[Jetson 上で Docker イメージをビルドするのが辛かったので EC2 上にビルド環境を作った - ABEJA Tech Blog](https://tech-blog.abeja.asia/entry/environment-of-building-docker-image-for-jetson)
[multiarch/qemu-user-static: `/usr/bin/qemu-*-static`](https://github.com/multiarch/qemu-user-static)
[docker + qemu で raspberry pi の開発環境構築 - Plamo Linux 日記](https://toshi-mtk.hatenablog.com/entry/2020/07/20/200643)
[ARM環境のRaspbianイメージをx86上のDockerで動かす - Qiita](https://qiita.com/hishi/items/61652e2d9755e17630de)
[QEMUのユーザーモードをDockerコンテナ上で使う - Qiita](https://qiita.com/FGtatsuro/items/c5dd8fdb028fe8948c2e)


# Jetson Nanoサーバー編

速いとは言わないけど手頃に動いてくれるJetson Nanoをサーバーにして、ディープラーニング等の勉強のための環境にしてしまおう、という企画。

## 手順

[※最新のDockerが動かない可能性（2021/11現在）](latest_docker.html)

### 前提

メインPCからSSH接続でJetson Nanoにアクセスするものとする。

### 準備

どこか適当に（自分の場合はホームディレクトリ）に専用のディレクトリを作成する。

~~~shell
$ mkdir ~/jupyterlab
$ cd ~/jupyterlab
~~~

以下のように中身を構成する

~~~
jupyterlab
├── docker
│   ├── Dockerfile
│   └── jupyter（空ディレクトリ）
├── docker-compose.yaml
└── work（空ディレクトリ）
~~~

命名適当すぎた。

* `docker`・・・コンテナイメージを作るためだったり、Jupyter Labの設定を入れるためのディレクトリ
* `docker-compose.yaml`・・・コンテナを立ち上げるためのファイル
* `work`・・・作ったファイルはここに保存

### `Dockerfile`の中身

~~~dockerfile
FROM nvcr.io/nvidia/l4t-pytorch:r32.6.1-pth1.9-py3

# Timezone
RUN apt update && apt install -y tzdata \
    && ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && apt clean \
    && rm -rf /var/lib/apt/lists/*

ENV TZ=Asia/Tokyo

# Packages (related to Jupyter Lab)
RUN python3 -m pip install --upgrade pip \
    && python3 -m pip install --no-cache-dir \
    jupyterlab \
    jupyterlab-git

# Packages (relatively large)
RUN python3 -m pip install --no-cache-dir \
    seaborn
~~~

自分は1行目を、[Jetson AI Specialist取得のためのプロジェクトで作ったコンテナ](https://github.com/take5553/face-check/blob/master/Dockerfile)で代用。

~~~dockerfile
FROM take5553/face-check:jp6.0
~~~

公式のPyTorchコンテナとの違いは

* GStreamerが使えるOpenCV
* Tkinter
* facenet-PyTorch
* JetCam

ぐらいなので公式のPyTorchからスタートしてもそこまで大きな問題にならないはず。問題になったら頑張れ。

### コンテナをビルド

~~~shell
$ cd docker
$ sudo docker build -t jupyterlab .
~~~

### コンテナを立ち上げ、設定ファイルを書き出す

~~~shell
$ sudo docker run -it --rm -v ~/jupyterlab/docker/jupyter:/root/.jupyter jupyterlab
~~~

これでコンテナの中に入り、以下のコマンドで設定ファイルを書き出す。

~~~shell
# jupyter lab --generate-config
~~~

書き出したらすぐにコンテナから出る。

~~~shell
# exit
~~~

設定ファイルを開く。

~~~shell
$ nano ~/jupyterlab/docker/jupyter/jupyter_lab_config.py
~~~

コメント部分をすべて無視して、末尾に以下を書く。

~~~python
import os
from IPython.lib import passwd
os.environ['JUPYTER_PASSWORD'] = passwd(os.environ['JUPYTER_PASSWORD'])

c.ServerApp.allow_password_change = False
c.ServerApp.allow_root = True
c.ServerApp.ip = '0.0.0.0'
c.ServerApp.password = os.environ['JUPYTER_PASSWORD']
c.ServerApp.password_required = True
c.ServerApp.root_dir = '/work'
~~~

### `docker-compose.yaml`の中身

~~~yaml
services:
    lab:
        build:
            context: ./docker
            dockerfile: Dockerfile
        restart: always
        environment:
          - LD_PRELOAD=/usr/lib/aarch64-linux-gnu/libgomp.so.1
          - JUPYTER_PASSWORD=(好きなパスワードにする)
        entrypoint: >
          jupyter lab
          --no-browser
        volumes:
          - ./docker/jupyter:/root/.jupyter
          - ./work:/work
        network_mode: "host"
        runtime: nvidia
~~~

これでコンテナを立ち上げる。

~~~shell
$ sudo docker-compose up
~~~

あとはブラウザから`(Jetson NanoのIP):8888`でアクセスすればOK。

終了方法はサーバーのコンソールに出力されているはず。

ブラウザから終了した場合はコンテナが終了しない可能性がある。コンテナが終了したかどうかはJetson Nano上で以下を打ち確認。

~~~shell
$ sudo docker ps
~~~

コンテナの情報が表示されたらコンテナがまだ残っているので、その場合は`docker-compose.yaml`があるディレクトリに移動して以下を打つ。

~~~shell
$ sudo docker-compose down
~~~

## 日常使用

1. Jetson Nanoを立ち上げ
2. `cd jupyterlab`
3. `sudo docker-compose up`
4. ブラウザから`(Jetson NanoのIP):8888`にアクセス
5. （作業）
6. サーバーを終了させる
7. コンテナが残ってないか`sudo docker ps`で確認。残っていたら`sudo docker-compose down`
8. `sudo shutdown -h now`でJetson Nanoを終了

## Pythonのパッケージを追加

### 1. とりあえず追加

Jupyter Lab上で以下を打てば良い。

~~~
!pip install (パッケージ名)
~~~

ただし、コンテナが削除されてしまうと当たり前だけどインストールされたパッケージは無くなる。

### 2. 永続的に追加

`Dockerfile`に追記して、コンテナをビルドし直せばよい。

## 参考

[memo: jupyterlab](https://zenn.dev/ningensei848/scraps/e2494c30e57c08)

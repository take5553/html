# セットアップとカメラの動作確認

## Dockerでセットアップ

Jetson Nano上で

~~~shell
$ git clone --recursive https://github.com/dusty-nv/jetson-inference
$ cd jetson-inference
$ docker/run.sh
~~~

これでDockerのイメージがダウンロードされる。途中で学習済みモデルをDLするかどうか聞かれるが、必要なときにダウンロードすればよいとのことなので、初回はデフォルトのままでよい。

## ソースからセットアップ

[jetson-inference/building-repo-2.md at master · dusty-nv/jetson-inference · GitHub](https://github.com/dusty-nv/jetson-inference/blob/master/docs/building-repo-2.md)

読んで何が起こっているのか分かればそれでいいと思う。結局Dockerでセットアップするのと同じ結果になるだけだし。勉強のためにやってもいいけど、結局書いてあることを追うだけで終わる可能性大。

ただ、ソースからセットアップすればDockerを毎回立ち上げる必要は無い。

## カメラの動作確認

### Jetson Nanoにモニターをつないでやっている場合

Dockerコンテナの中から（ソースからセットアップした場合はJetson Nano上のどこでも）以下を打つ。

~~~shell
$ video-viewer /dev/video0
~~~

カメラ画像を表示するウィンドウが現れるはず。VNC接続でモニターしている場合も同様。

### SSH接続等リモートからヘッドレス状態でやっている場合

Dockerコンテナの中から以下を打つ。

~~~shell
$ video-viewer /dev/video0 rtp://(ローカルPCのIP):1234
~~~

これでRTPというプロトコルでローカルPCに映像を飛ばし続けるので、これを受けてあげれば良い。

説明ビデオの中ではGStreamerがオススメされている。自分のPCにも元々入っていたみたいで、インストール無しで以下が通った。ちなみに`1234`ポートを開けるか、他のポートを使うかしないといけない。

~~~shell
$ gst-launch-1.0 -v udpsrc port=1234 \
caps = "application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)96" ! \
rtph264depay ! decodebin ! videoconvert ! autovideosink
~~~

RTPをサポートするビュワーなら何でもいいらしい。VLCを使ったやり方も紹介されていた。

[jetson-inference/aux-streaming.md at master · dusty-nv/jetson-inference · GitHub](https://github.com/dusty-nv/jetson-inference/blob/master/docs/aux-streaming.md#rtp)

## 関連

`video-viewer`に渡せるパラメーターは以下を参照。

[jetson-inference/aux-streaming.md at master · dusty-nv/jetson-inference · GitHub](https://github.com/dusty-nv/jetson-inference/blob/master/docs/aux-streaming.md)

実際は

* `video-viewer`
* `imagenet`
* `detectnet`

に共通して使えるパラメーターらしい。

Input StreamsとOutput Streamsのところは特に重要。これを組み合わせれば、「入力：カメラの映像、出力：動画ファイル」とか「入力：リモート機器からのRTP（またはRTSP）ストリーム、出力：別のリモート機器へのRTP出力」ができるっぽい。

USBカメラはV4L2デバイスとしてLinuxに認識されるものだったら良いとのことだけど、それを確認するには`v4l2-ctl`を使う。

Jetson Nano上（コンテナ内では無い）で以下を打つ。

~~~shell
$ sudo apt install v4l-utils
$ v4l2-ctl --list-devices
~~~

すると以下が出力される。

~~~
HD Webcam eMeet C960 (usb-70090000.xusb-2):
	/dev/video0
~~~

これで出力されたら認識されているということ。`v4l2-ctl`を使えばカメラの情報を色々と見ることができるらしい。詳しくは以下。

参考：[Linux: 利用できるWebカメラの情報を取得する](https://leico.github.io/TechnicalNote/Linux/webcam-usage)

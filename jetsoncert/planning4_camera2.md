# 調査4　GStreamer

マルチメディアフレームワーク。今回のケースで言えばVideo4Linux(V4L)のAPIをソースとし、モニターやOpenCVに出力する役割を担う。

※OpenCVはGStreamerを介さなくても映像をキャプチャーできるが、GStreamerを介すことで細かいことができるようになる。

参考：[GStreamer アプリケーション開発マニュアル 日本語訳 (0.10.25.1)](http://oss.infoscience.co.jp/gstreamer/index.html)

参考：[gst-launch-1.0](https://gstreamer.freedesktop.org/documentation/tools/gst-launch.html?gi-language=c)

## インストール

今回使うコンテナでの環境は以下。

~~~shell
$ sudo apt install \
	gstreamer1.0-alsa \
    gstreamer1.0-libav \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-tools \
	libgstreamer1.0-dev \
	libgstreamer-plugins-base1.0-dev
~~~

他にも色々あるらしい。

~~~shell
$ sudo apt install \
	gstreamer1.0-doc \
	gstreamer1.0-tools \
	gstreamer1.0-x \
	gstreamer1.0-gl \
	gstreamer1.0-gtk3 \
	gstreamer1.0-qt5 \
	gstreamer1.0-pulseaudios
~~~

## テスト

真っ当な環境（？）なら普通に以下でテスト画像が表示される。右下のノイズが動いているはずなので、動画であることが分かる。

~~~shell
$ gst-launch-1.0 videotestsrc ! autovideosink
~~~

![image-20210815164709447](image/planning4_camera2/image-20210815164709447.png)

自分のように

~~~
ローカルPC > Jetson Nano > コンテナ
~~~

という接続で、コンテナ内で起動させてローカルPCに表示させたい場合は`autovideosink`ではなく`ximagesink`を指定する。ただし、

* [コンテナ内で起動したXアプリがローカルPCで起動する設定をしておく](../docker/l4t_base1.html)
* `gstreamer1.0-x`をインストールしておく

~~~shell
$ gst-launch-1.0 videotestsrc ! ximagesink
~~~

また、`videotestsrc`には`pattern`という引数があって、

~~~shell
$ gst-launch-1.0 videotestsrc pattern=18 ! ximagesink
~~~

とすると、ボールが動く映像が映る。

## V4Lをソースにする（USBカメラ）

~~~shell
$ gst-launch-1.0 v4l2src device=/dev/video0 ! ximagesink
~~~

~~~
Setting pipeline to PAUSED ...
Pipeline is live and does not need PREROLL ...
Setting pipeline to PLAYING ...
ERROR: from element /GstPipeline:pipeline0/GstV4l2Src:v4l2src0: Internal data stream error.
Additional debug info:
gstbasesrc.c(3055): gst_base_src_loop (): /GstPipeline:pipeline0/GstV4l2Src:v4l2src0:
streaming stopped, reason not-negotiated (-4)
Execution ended after 0:00:00.000345216
Setting pipeline to PAUSED ...
Setting pipeline to READY ...
Setting pipeline to NULL ...
Freeing pipeline ...
~~~

`not-negotiated (-4)`が出ると「上手くパイプラインが繋がっていない」という意味っぽい。

`videoconvert`を挟むとカメラ映像を取得することができる。

~~~shell
$ gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert ! ximagesink
~~~

ただ、最大解像度で取り込んでるので解像度を調整する。

~~~shell
$ gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,width=160,height=120 ! videoconvert ! ximagesink
~~~

設定できる解像度は`v4l2-ctl -d 0 --list-formats-ext`で確認できるものだけらしい。

ちなみに`videoconvert`が入ると、`v4l2-ctl -V`で見れる情報も書き換わる。

~~~shell
$ v4l2-ctl -V
~~~

~~~
Format Video Capture:
Width/Height      : 1920/1080
Pixel Format      : 'YUYV'
Field             : None
Bytes per Line    : 3840
Size Image        : 4147200
Colorspace        : sRGB
Transfer Function : Default (maps to sRGB)
YCbCr/HSV Encoding: Default (maps to ITU-R 601)
Quantization      : Default (maps to Limited Range)
Flags             :
~~~

どうも`video/x-raw`を指定すると`Pixel Format : 'YUYV'`になるらしい。

## MJPG（Motion-JPEG）を指定する（USBカメラ）

~~~shell
$ gst-launch-1.0 v4l2src device=/dev/video0 ! jpegdec ! videoconvert ! ximagesink
~~~

これも最大解像度になってしまうので、以下のようにして調整する。

~~~shell
$ gst-launch-1.0 v4l2src device=/dev/video0 ! image/jpeg,width=800,height=600 ! jpegdec ! videoconvert ! ximagesink
~~~

同じ解像度でも、`video/x-raw`で指定し`jpegdec`無しで繋いだときとはFPSが違う。これはそもそもカメラの性能がそうなっている。

~~~shell
$ v4l2-ctl -d 0 --list-formats-ext
~~~

~~~
ioctl: VIDIOC_ENUM_FMT
    Index       : 0
    Type        : Video Capture
    Pixel Format: 'MJPG' (compressed)
    Name        : Motion-JPEG
    
		(略)
		
        Size: Discrete 800x600
        Interval: Discrete 0.033s (30.000 fps)
        
		(略)

    Index       : 1
    Type        : Video Capture
    Pixel Format: 'YUYV'
    Name        : YUYV 4:2:2

		(略)

        Size: Discrete 800x600
        Interval: Discrete 0.100s (10.000 fps)

		(略)
~~~

## nVIDIA製のプラグイン（CSIカメラ）

Jetson Nanoに入っている、GStreamerで使えるプラグイン。全容は以下を参照。

https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html#

動いた例1（Raspberry Pi Camera Module v2使用）（[参考](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html#wwpID0E0FR0HA)）

~~~
$ gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM), width=(int)1280, height=(int)720, format=(string)NV12, framerate=(fraction)30/1' ! nvvidconv ! ximagesink
~~~

動いた例2（Raspberry Pi Camera Module v2使用）（[前処理でCUDAを使うらしい](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html#wwpID0E0BI0HA)）

~~~
$ gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM), width=(int)1280, height=(int)720, format=(string)NV12, framerate=(fraction)30/1' ! nvivafilter cuda-process=true customer-lib-name="libnvsample_cudaprocess.so" !  'video/x-raw(memory:NVMM), format=(string)NV12' ! nvvidconv ! ximagesink
~~~

1. ディスプレイの大きさ以上の解像度を指定するとエラーが起きるらしい。

2. `nvvidconv`を通せば何でもいける気がする。
3. 他の`~~~sink`もあるけど、~~今の所`ximagesink`しか成功しない。~~GUI上でやったら`nv3dsink`、`nveglglessink`でもテスト映像を映すことに成功した。

素晴らしく素晴らしい解説をしているサイトを見つけた。

[NVIDIA JetsonのGStreamerでの4画面表示 – senooken.jp](https://senooken.jp/post/2020/11/18/)

`(memory:NVMM)`の意味。

[What is the difference between 'video/x-raw(memory:NVMM)' and video/x-raw? - Jetson & Embedded Systems / Jetson TX2 - NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/what-is-the-difference-between-video-x-raw-memory-nvmm-and-video-x-raw/72424)

[DMA対応と言われたら（1） | 学校では教えてくれないこと | [技術コラム集]組込みの門 | ユークエスト株式会社](https://www.uquest.co.jp/embedded/learning/lecture15-1.html)

## nVIDIA製のプラグインで画面の切り取り（CSIカメラ）

~~~
$ gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM), width=(int)1280, height=(int)720, format=(string)NV12, framerate=(fraction)30/1' ! nvvidconv flip-method=2 left=320 right=960 top=0 bottom=720 ! 'video/x-raw(memory:NVMM), width=(int)600, height=(int)720, format=(string)NV12, framerate=(fraction)30/1' ! nv3dsink -e
~~~

1. `nvarguscamerasrc`
2. フォーマット情報でカメラソースの解像度を指定
3. `nvvidconv`で「上下反転（Raspberry Piカメラモジュールは天地逆さまのキャプチャーがデフォなので）」をしてから、「X軸`320`から`960`の範囲、Y軸`0`から`720`の範囲（つまり全体）を切り取り」をする
4. フォーマット情報で出力映像の解像度を指定（受け取った解像度と出力解像度が一致しない場合、勝手に拡大縮小するらしい）
5. `nv3dsink`で映像出力。（`-e`はパイプライン終了の合図）

`nvvidconv`のCropping（切り取り）の指定がGStreamer標準の`videocrop`のやり方と違って混乱した。

* `nvvidconv`

  左右：`left`の座標から`right`の座標までの範囲で切り取り。

  上下：`top`の座標から`bottom`の座標までの範囲で切り取り。

* `videocrop`

  左：`left`のピクセル分切り取り

  右：`right`のピクセル分切り取り

  上：`top`のピクセル分切り取り

  下：`bottom`のピクセル分切り取り

また、`nvvidconv`で切り取りした後はフォーマット指定で切り取り幅を明示してあげないと勝手に拡大縮小と判断してしまう。

## USBカメラでも`nvarguscamerasrc`は使えないのか

使えないらしい。

https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3231/index.html#page/Tegra%2520Linux%2520Driver%2520Package%2520Development%2520Guide%2Fjetson_xavier_camera_soft_archi.html%23wwpID0E0AC0HA

CSIカメラでは`v4l2src`と`nvarguscamerasrc`のどちらでも取り込めるらしいけど、`nvarguscamerasrc`を使いつつ途中も`nv~`系のプラグインを良いような雰囲気のことを言っている人が居た（かなり曖昧）。

https://forums.developer.nvidia.com/t/gstreamer-nvarguscamerasrc-and-v4l2src-elements-comparison-on-flowchart/74875/3

## 参考リンク

[GStreamer Advent Calendar 2015 - Qiita](https://qiita.com/advent-calendar/2015/gstreamer)
[GStreamer Advent Calendar 2016 - Qiita](https://qiita.com/advent-calendar/2016/gstreamer)

[cropping image using nvvidconv - Jetson & Embedded Systems / Jetson TX1 - NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/cropping-image-using-nvvidconv/48489)

[【第1回】色の世界 (1/3)：CodeZine（コードジン）](https://codezine.jp/article/detail/3749)（無料の会員登録が必要だけどPixel Formatの基礎知識として有用）
[YUVをちゃんと理解してからRGBにコンバートしましょうね | Technology | KLablog | KLab株式会社](https://www.klab.com/jp/blog/tech/2016/1054828175.html)
[YUV - Wikipedia](https://ja.wikipedia.org/wiki/YUV)

https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3231/index.html#page/Tegra%2520Linux%2520Driver%2520Package%2520Development%2520Guide%2Fjetson_xavier_camera_soft_archi.html%23wwpID0E0OC0HA

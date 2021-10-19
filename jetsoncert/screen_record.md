# スクリーンを動画に撮る

いわゆるスクリーンキャプチャーだけど、それをGStreamerを使ってコマンドラインでやってしまおうという企画。特にソフトをインストールする必要が無いから楽。

## 基本

参考：[How do I screen record on the Jetson TX2 - Jetson & Embedded Systems / Jetson TX2 - NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/how-do-i-screen-record-on-the-jetson-tx2/178956/5)

~~~
gst-launch-1.0 ximagesrc use-damage=0 \
    ! video/x-raw \
    ! nvvidconv \
    ! 'video/x-raw(memory:NVMM),format=NV12' \
    ! nvv4l2h264enc \
    ! h264parse \
    ! matroskamux \
    ! filesink location=a.mkv
~~~

ファイル形式が`.mkv`（[Matroska - Wikipedia](https://ja.wikipedia.org/wiki/Matroska)）になっているけどちゃんとファイルに保存される。

どうやらこれは以下のように前半と後半に分けられる。

~~~
gst-launch-1.0 ximagesrc use-damage=0 \
    ! video/x-raw \
    ! nvvidconv \
    ! 'video/x-raw(memory:NVMM),format=NV12' \
    ! nvv4l2h264enc \
    ! h264parse \
    
--------------------------------------    
    
    ! matroskamux \
    ! filesink location=a.mkv
~~~

前半ではスクリーンをキャプチャーしてh.264で圧縮、後半でコンテナに格納しファイル化。

## mp4化

~~~
gst-launch-1.0 -e ximagesrc use-damage=0 \
    ! video/x-raw \
    ! nvvidconv \
    ! 'video/x-raw(memory:NVMM),format=NV12' \
    ! nvv4l2h264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mp4mux \
    ! filesink location=a.mp4
~~~

`h264parse`から後を変更。主には`mp4mux`でmp4コンテナに格納してファイル化するようにしただけ。`-e`オプションをつけて`ctrl-C`を押したときにEOSイベントを発行するようにした。

参考：[Events](https://gstreamer.freedesktop.org/documentation/additional/design/events.html?gi-language=c#eos)

## 音声を追加

~~~
gst-launch-1.0 -e ximagesrc use-damage=0 \
    ! video/x-raw \
    ! nvvidconv \
    ! 'video/x-raw(memory:NVMM),format=NV12' \
    ! nvv4l2h264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mp4mux name=mux \
    ! filesink location=a.mp4 \
    audiotestsrc \
    ! voaacenc \
    ! mux.
~~~

muxエレメントに`name`プロパティをつけて、どこか適当なところで`audiotestsrc`から始まるパイプラインを開始して`mux.`で終わる。

これは以下のように整理できるらしい。

~~~
gst-launch-1.0 -e \
    ximagesrc use-damage=0 \
    ! video/x-raw \
    ! nvvidconv \
    ! 'video/x-raw(memory:NVMM),format=NV12' \
    ! nvv4l2h264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mux.video_0 \
    audiotestsrc \
    ! voaacenc \
    ! mux.audio_0 \
    mp4mux name=mux \
    ! filesink location=a.mp4 \
~~~

なるほど。これで、映像部分と音声部分でわかりやすくなった。

## 音声ソースをWebカメラのものに変更

~~~
gst-launch-1.0 -e \
    ximagesrc use-damage=0 \
    ! video/x-raw \
    ! nvvidconv \
    ! 'video/x-raw(memory:NVMM),format=NV12' \
    ! nvv4l2h264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mux.video_0 \
    pulsesrc device=alsa_input.usb-SHENZHEN_EMEET_TECHNOLOGY_CO._LTD._HD_Webcam_eMeet_C960_Ucamera002-02.analog-mono \
    ! lamemp3enc \
    ! mux.audio_0 \
    mp4mux name=mux \
    ! filesink location=a.mp4 \
~~~

音声ソースエレメントを`pulsesrc`に変えて、デバイスをWebカメラのものを指定。エンコードはAACでは音飛びが激しかったのでMP3を指定。

Webカメラのマイクは以下のようにして取得可能。

~~~shell
$ pactl list | grep alsa_input
~~~

`pactl`はPulseAudio controlのことらしい。深追いはやめとこ。

とりあえずWebカメラ以外のマイクを持っていたとしても同じ方法で調べれば出てくるということか。

参考：[pactl(1) - Linux man page](https://linux.die.net/man/1/pactl)


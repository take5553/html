# スクリーンを動画に撮る2

前回のやり方は「Jetson Nano上で録画する」だったけど、やはりというか、マシンパワーが気になるところなので、素直に「メインPCでJetson NanoのHDMI出力をキャプチャーする」という方法に切り替える。

ちなみに自分のメインPCはLinux（Arch Linux）なのであしからず。あと、GStreamerが入っていること前提。

## 準備

HDMIキャプチャーカードを購入

[Amazon.co.jp: KeeQii HD HDMI キャプチャーボード USB3.0 1080P HDMI ゲームキャプチャー、 ビデオキャプチャカード ゲーム実況生配信、画面共有、録画、ライブ会議に適用 DSLRビデオカメラ Nintendo Switch、Xbox One、OBS Studio XSplit対応 電源不要（シルバー） : パソコン・周辺機器](https://www.amazon.co.jp/KeeQii-%E3%82%B2%E3%83%BC%E3%83%A0%E3%82%AD%E3%83%A3%E3%83%97%E3%83%81%E3%83%A3%E3%83%BC%E3%80%81-%E3%83%93%E3%83%87%E3%82%AA%E3%82%AD%E3%83%A3%E3%83%97%E3%83%81%E3%83%A3%E3%82%AB%E3%83%BC%E3%83%89-%E3%82%B2%E3%83%BC%E3%83%A0%E5%AE%9F%E6%B3%81%E7%94%9F%E9%85%8D%E4%BF%A1%E3%80%81%E7%94%BB%E9%9D%A2%E5%85%B1%E6%9C%89%E3%80%81%E9%8C%B2%E7%94%BB%E3%80%81%E3%83%A9%E3%82%A4%E3%83%96%E4%BC%9A%E8%AD%B0%E3%81%AB%E9%81%A9%E7%94%A8-Switch%E3%80%81Xbox/dp/B09BJCR7XD)

メインPCのUSBに挿し、Jetson NanoからのHDMI入力を受け取る。

## 調査

キャプチャーカードを挿した状態で、まずはどういう扱いになっているのか調べる。

~~~shell
$ v4l2-ctl --list-devices
~~~

~~~
USB3. 0 capture: USB3. 0 captur (usb-0000:00:14.0-3):
        /dev/video0
        /dev/video1
        /dev/media0
~~~

ほう。

~~~shell
$ v4l2-ctl -d /dev/video0 --info
~~~

~~~
Driver Info:
	Driver name      : uvcvideo
	Card type        : USB3. 0 capture: USB3. 0 captur
	Bus info         : usb-0000:00:14.0-3
	Driver version   : 5.12.15
	Capabilities     : 0x84a00001
		Video Capture
		Metadata Capture
		Streaming
		Extended Pix Format
		Device Capabilities
	Device Caps      : 0x04200001
		Video Capture
		Streaming
		Extended Pix Format
Media Driver Info:
	Driver name      : uvcvideo
	Model            : USB3. 0 capture: USB3. 0 captur
	Serial           : 
	Bus info         : usb-0000:00:14.0-3
	Media version    : 5.12.15
	Hardware revision: 0x00002100 (8448)
	Driver version   : 5.12.15
Interface Info:
	ID               : 0x03000002
	Type             : V4L Video
Entity Info:
	ID               : 0x00000001 (1)
	Name             : USB3. 0 capture: USB3. 0 captur
	Function         : V4L2 I/O
	Flags         : default
	Pad 0x01000007   : 0: Sink
	  Link 0x0200000d: from remote pad 0x100000a of entity 'Processing 2': Data, Enabled, Immutable
~~~

ほほう。`uvcvideo`ということは普通のWebカメラと同じように取り込めるということか。`/dev/video1`も似たような情報を返した。

`/dev/media0`について。

~~~shell
$ v4l2-ctl -d /dev/media0 --info
~~~

~~~
Unable to detect what device /dev/media0 is, exiting.
~~~

これは音声入力かな？V4LはVideo for Linuxだから映像に特化してるとか？よく分からん。

[前回](screen_record.html)のやり方を参考にして音声デバイスを調べてみる。

~~~shell
$ pactl list | grep alsa_input
~~~

~~~
	名前: alsa_input.usb-MACROSILICON_USB3._0_capture-02.analog-stereo
~~~

おお、これっぽい。

映像の対応解像度を見てみる。

~~~shell
$ v4l2-ctl -d /dev/video0 --list-formats-ext
~~~

~~~
ioctl: VIDIOC_ENUM_FMT
	Type: Video Capture

	[0]: 'MJPG' (Motion-JPEG, compressed)
		Size: Discrete 1920x1080
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1600x1200
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1360x768
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1280x1024
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1280x960
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1024x768
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 800x600
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 720x576
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 720x480
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
	[1]: 'YUYV' (YUYV 4:2:2)
		Size: Discrete 1920x1080
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1600x1200
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1360x768
			Interval: Discrete 0.125s (8.000 fps)
		Size: Discrete 1280x1024
			Interval: Discrete 0.125s (8.000 fps)
		Size: Discrete 1280x960
			Interval: Discrete 0.125s (8.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1024x768
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 800x600
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 720x576
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 720x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)

~~~

なるほど。Motion JPEGにすれば60fpsで取り込めるのか。

## メインPCでJetson Nanoの映像を取り込む

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! xvimagesink
~~~

とりあえず映像を取り込む。Jetson側でも解像度を設定してあげる。

## メインPCでJetson Nanoの音声を取り込む

~~~shell
$ gst-launch-1.0 -e \
    pulsesrc device=alsa_input.usb-MACROSILICON_USB3._0_capture-02.analog-stereo \
    ! autoaudiosink
~~~

音声の取り込み。これで一応音が出る。

## Jetson Nanoの映像＋音声

これはソースエレメントはいくつ書いてもいいので、単純に書き並べるだけ。

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! xvimagesink \
    pulsesrc device=alsa_input.usb-MACROSILICON_USB3._0_capture-02.analog-stereo \
    ! autoaudiosink
~~~

## 映像を分岐させる

モニターで確認しつつ動画ファイルに保存ということができたら最高なので、とりあえず分岐だけやってみる。

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! tee name=t \
    ! queue \
    ! xvimagesink \
    t. \
    ! queue \
    ! xvimagesink
~~~

これで同じ映像が2ウィンドウに分岐する。

`tee`はLinuxコマンドの`tee`と同じで、パイプラインを分岐させる役割を持つ。分岐は何個あっても良いらしい。ただし、`tee`の後は`queue`で受けてあげないといけない。

イメージはこんな感じ。

~~~
jpegdec → tee → queue → xvimagesink
           ↓
           t. → queue → xvimagesink
~~~

3分岐以上もできる。

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! tee name=t \
    ! queue \
    ! xvimagesink \
    t. \
    ! queue \
    ! xvimagesink
    t. \
    ! queue \
    ! xvimagesink
~~~

イメージは以下。

~~~
jpegdec → tee → queue → xvimagesink
           ↓
           t. → queue → xvimagesink
           ↓
           t. → queue → xvimagesink
~~~

## 映像を取り込みつつ動画ファイルに保存する（音声無し）

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! tee name=t \
    ! queue \
    ! xvimagesink \
    t. \
    ! queue \
    ! videoconvert \
    ! nvh264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mp4mux \
    ! filesink location=a.mp4
~~~

メインPCでは`nvv4l2h264enc`がインストールされていなかったので代わりに`nvh264enc`を使用した。本当にちゃんと使えてるのかどうかは知らんけど。

## 映像に音声を合成して表示しつつ動画ファイルに保存する

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! tee name=videotee \
    ! queue \
    ! videoconvert \
    ! nvh264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mux.video_0 \
    pulsesrc device=alsa_input.usb-MACROSILICON_USB3._0_capture-02.analog-stereo \
    ! tee name=audiotee \
    ! queue \
    ! lamemp3enc \
    ! mux.audio_0 \
    mp4mux name=mux \
    ! filesink location=a.mp4 \
    videotee. \
    ! queue \
    ! xvimagesink \
    audiotee. \
    ! queue \
    ! decodebin \
    ! audioconvert \
    ! autoaudiosink
~~~

一応これでできた。最後の`audiotee.`からの`decodebin`と`audioconvert`はいらんやろと思いつつ、外すとうまく動かなかった。なんでや。

たださすがに遅延が若干あって、マウスの動きが明らかに遅れる。

## HDMI分配器でなんとかする

Jetson Nanoの映像を分配してモニターとPCに送り、PCでは録画に専念することとした。

[Amazon | Vikrin HDMI 分配器 1入力2出力 HDMI スプリッター PSE認証済みアダプタ付き PS4 pro/PS3/Xbox/任天堂スイッチ/BDレコーダーなど対応4K 3D 1080P HDCP1.4対応 日本語説明書付き | Vikrin | AVセレクター](https://www.amazon.co.jp/Vikrin-PSE%E8%AA%8D%E8%A8%BC%E6%B8%88%E3%81%BF%E3%82%A2%E3%83%80%E3%83%97%E3%82%BF%E4%BB%98%E3%81%8D-BD%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%80%E3%83%BC%E3%81%AA%E3%81%A9%E5%AF%BE%E5%BF%9C4K-HDCP1-4%E5%AF%BE%E5%BF%9C-%E6%97%A5%E6%9C%AC%E8%AA%9E%E8%AA%AC%E6%98%8E%E6%9B%B8%E4%BB%98%E3%81%8D/dp/B09CZB1BJS)

GStreamerのパイプラインは、分岐を減らせばいいだけ。

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! videoconvert \
    ! nvh264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mux.video_0 \
    pulsesrc device=alsa_input.usb-MACROSILICON_USB3._0_capture-02.analog-stereo \
    ! lamemp3enc \
    ! mux.audio_0 \
    mp4mux name=mux \
    ! filesink location=a.mp4
~~~

## マイクの音を合成する

これ買った。

[Amazon | ソニー エレクトレットコンデンサーマイクロホン PC/ゲーム用 PCV80U ECM-PCV80U | 楽器・音響機器 | 楽器・音響機器](https://www.amazon.co.jp/SONY-%E3%82%A8%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AC%E3%83%83%E3%83%88%E3%82%B3%E3%83%B3%E3%83%87%E3%83%B3%E3%82%B5%E3%83%BC%E3%83%9E%E3%82%A4%E3%82%AF%E3%83%AD%E3%83%9B%E3%83%B3-%E3%82%B2%E3%83%BC%E3%83%A0%E7%94%A8-PCV80U-ECM-PCV80U/dp/B005M2HDA6)

デバイス名は以下。

~~~shell
$ pactl list | grep alsa_input
~~~

~~~
	名前: alsa_input.usb-Sony_Corporation_UAB-80-00.mono-fallback
~~~

パイプライン上では`audiomixer`を使えば良い。

~~~shell
$ gst-launch-1.0 -e \
    v4l2src device=/dev/video0 \
    ! image/jpeg,width=1024,height=768,framerate=60/1 \
    ! jpegdec \
    ! videoconvert \
    ! nvh264enc \
    ! h264parse \
    ! video/x-h264,stream-format=avc,alignment=au \
    ! mux.video_0 \
    pulsesrc device=alsa_input.usb-Sony_Corporation_UAB-80-00.mono-fallback \
    ! audiomix. \
    pulsesrc device=alsa_input.usb-MACROSILICON_USB3._0_capture-02.analog-stereo \
    ! audiomixer name=audiomix \
    ! lamemp3enc \
    ! mux.audio_0 \
    mp4mux name=mux \
    ! filesink location=a.mp4
~~~

## 参考

[Raspberry Pi で HDMI キャプチャして RTMP で放送する | 備忘録](https://blog.akagi.jp/archives/4937.html)
[格安USB-HDMIキャプチャーボードを使ってノートPCをラズパイ・Jetson・Nintendo Switchのディスプレイにする方法 - karaage. [からあげ]](https://karaage.hatenadiary.jp/entry/2021/01/04/073000)
[HDMI入力をRaspberry Piで駆使する | 犬アイコンのみっきー](https://www.mzyy94.com/blog/2020/04/10/raspberrypi-hdmi-input/)


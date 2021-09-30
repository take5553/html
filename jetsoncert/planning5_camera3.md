# 調査5　Jetson Nano用カメラライブラリ（Python）

## 候補

### JetCam

[NVIDIA-AI-IOT/jetcam: Easy to use Python camera interface for NVIDIA Jetson](https://github.com/NVIDIA-AI-IOT/jetcam)

こちらを採用する。

### NanoCamera

[thehapyone/NanoCamera: A simple to use camera interface for the Jetson Nano for working with USB and CSI cameras in Python.](https://github.com/thehapyone/NanoCamera)

## 改造

デフォルトのJetCamはあまり込み入った調整ができないので改造する。

~~~python
from jetcam.usb_camera import USBCamera


class MyCamera(USBCamera):

    def __init__(self, *args, **kwargs):
        super(MyCamera, self).__init__(*args, **kwargs)


    def _gst_str(self):
        settings = (同じディレクトリにあるsettings.jsonを読み込んで設定を辞書として返す何かのメソッド)
        return settings['gst_str']
    
    
    def _read(self):
        re, image = self.cap.read()
        if re:
            return image
        else:
            raise RuntimeError('Could not read image from camera')
~~~

[`USBCamera`](https://github.com/NVIDIA-AI-IOT/jetcam/blob/master/jetcam/usb_camera.py)を継承して、GStreamerパイプラインを構成する文字列を返すメソッドを書き換える。具体的には`settings.json`の中にGStreamerパイプラインを書き込んで保存し、それを使うことで柔軟に設定を変えられるようにする。

さらに、デフォルトの`_read`メソッドでは表示サイズに合わせてリサイズしていたけれど、それもGStreamerに任せてしまえばいいので、変にリサイズしないようにする。

# POSTURE ANALYSIS APPLICATION USING JETSON NANO

カメラ画像から姿勢を分析して画像を保存する。

[GitHub](https://github.com/ysfzkn/posture-anaylsis-app)　[Youtube](https://www.youtube.com/watch?v=bQAjxHcxU6A)

![https://user-images.githubusercontent.com/58569590/113511234-89bd8080-9567-11eb-8f77-f8bfb8b6c155.jpg](image/community-project-02/113511234-89bd8080-9567-11eb-8f77-f8bfb8b6c155.jpg)

## 概要

* FlaskでWebアプリという形にしてJetson内で動かしている。なのでクライアント（Winなど）からリモートでアクセスして結果を得る
* [OpenPose](https://github.com/CMU-Perceptual-Computing-Lab/openpose)という外部APIを使用して姿勢をキャプチャーしている
  * OpenPoseは少なくとも2.5GBのRAMが必要なので、Jetson Nano (2GB)ではちょっと微妙。
  * とにかく使って得られた座標をとにかくPythonで描画している

## 感想

* OpenPoseをJetsonで動かして、それをWebサーバーに見立ててアクセスすればいいんじゃね的な発想。
* 精度は？
* 「使える！」（どんな風に使えるとは言っていない）

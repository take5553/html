# NVIDIA L4T PyTorchイメージ

自分としてはここに来るのがDocker編の最終目標だったりして。

## 概要

PyTorchが使えるコンテナ。公式は[ここ](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-pytorch)。

* Baseがベースなので、Baseでできることはできる。
* Python 3.6
* PyTorch、torchvision、torchaudioが入っている。（ただし使用するバージョンはJetPackのバージョンにより違う）

[Dockerfileを見る](https://github.com/dusty-nv/jetson-containers/blob/master/Dockerfile.pytorch)ともう少し細かく何が入っているのか把握できる。PILLOWとPyCUDAも入っているらしい。

## 立ち上げ

面倒なので最初から`docker-compose.yml`を書いちゃう。`~/my-docker/pytorch`ディレクトリを作ってそこに以下を作成。

~~~yaml
services:
    test:
        image: nvcr.io/nvidia/l4t-pytorch:r32.5.0-pth1.7-py3
        runtime: nvidia
        environment:
          - DISPLAY=$DISPLAY
        volumes:
          - /tmp/.X11-unix:/tmp/.X11-unix
          - ~/.Xauthority:/root/.Xauthority
        network_mode: "host"
        tty: true
        stdin_open: true
~~~

`l4t-pytorch:r32.5.0-pth1.7-py3`の部分は使っているJetPackのバージョンに合わせる。詳しくは[こちら](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-pytorch)。

立ち上げ。

~~~shell
$ cd ~/my-docker/pytorch
$ sudo docker-compose up -d
$ sudo docker attach pytorch_test_1
~~~

PyTorchの確認。

~~~shell
# python3
~~~

~~~python
>>> import torch
>>>
~~~

エラーが出なかったのでOK。

もう少し色々やってみる。（[参考](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048)）

~~~python
>>> print(torch.__version__)
1.7.0
>>> print('CUDA available: ' + str(torch.cuda.is_available()))
CUDA available: True
>>> print('cuDNN version: ' + str(torch.backends.cudnn.version()))
cuDNN version: 8000
>>> a = torch.cuda.FloatTensor(2).zero_()
>>> print('Tensor a = ' + str(a))
Tensor a = tensor([0., 0.], device='cuda:0')
>>> b = torch.randn(2).cuda()
>>> print('Tensor b = ' + str(b))
Tensor b = tensor([ 1.6762, -1.5597], device='cuda:0')
>>> c = a + b
>>> print('Tensor c = ' + str(c))
Tensor c = tensor([ 1.6762, -1.5597], device='cuda:0')
~~~

~~~python
>>> import torchvision
>>> print(torchvision.__version__)
0.8.0a0+45f960c
~~~

ほうほう。

## まとめ

前提としては「[ここ](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048)に従えばaarch64（Jetson Nanoが使うCPUアーキテクチャ）上で動くPyTorchを直接JetPackにインストールできる」とした上で、ミスるかもしれないなぁって人向けにDockerのイメージを作ってあげたよってこと。

このことを理解するのにどんだけ時間かけんねん俺。

## おまけ

[Jetson AI Certification編](../jetsoncert/)のGetting Started with AI on Jetson Nanoコースで使った[コンテナ](https://ngc.nvidia.com/catalog/containers/nvidia:dli:dli-nano-ai)は、このPyTorchコンテナに追加で

* JupyterLab
* Jetcam
* その他必要なもの（Node.js、libffi-dev、libssl1.0-dev）

を入れたものだったということが分かった。そうかそういうことか。


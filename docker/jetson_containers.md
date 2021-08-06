# 公式イメージ

nVIDIAが公開している公式イメージは[ここ](https://ngc.nvidia.com/catalog/containers)にあって、検索BOXに`jetson`と入れて検索するとJetsonで使えるDockerイメージが出てくる。Demoを除けば、開発用途としては以下。

* [NVIDIA L4T Base](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-base)
* [NVIDIA L4T PyTorch](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-pytorch)
* [NVIDIA L4T TensorFlow](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-tensorflow)
* [NVIDIA L4T ML](https://ngc.nvidia.com/catalog/containers/nvidia:l4t-ml)
* [DeepStream-l4t](https://ngc.nvidia.com/catalog/containers/nvidia:deepstream-l4t)

名前から分かるとおり、Baseがベースになって、PyTorchやTensorFlowがベースから追加でインストールされたもの、そしてそのどっちもが含まれているのがML（Machine Leaning）となっている。DeepStreamは謎。

とりあえずシンプルな考えとしては

「NVIDIA L4T PyTorchコンテナを立ち上げて中に入ればPyTorchが使える！」

となる。

ただ、Baseコンテナの説明文にはDisplay（つまりGUI）に関する記述があるし、Baseコンテナでどんなことができるのかを知ればPyTorchコンテナでも使えるはず。少しBaseコンテナを触ってみる。


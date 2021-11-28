# 最新のDockerが動かない可能性

2021/11月現在

## 症状

`runtime`引数に`nvidia`と指定してコンテナを動かそうとすると、

~~~shell
$ sudo docker run --runtime nvidia (その他引数)
~~~

以下のようなエラーが出る。

~~~
docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process caused: error adding seccomp filter rule for syscall clone3: permission denied: unknown.
~~~

`--runtime nvidia`の指定を外すとコンテナが立ち上がるけど、それでは意味が無い。

## 原因

最新バージョンのバグらしい。（2021/11月現在）

~~~shell
$ docker -v
Docker version 20.10.7, build 20.10.7-0ubuntu5~18.04.3
~~~

詳しい議論は[こちら](https://github.com/containerd/containerd/issues/6203)（英語）

## 解決策

### 1. `nvidia-container-toolkit`を更新する

※2021/11/28現在の最新解決策

詳しい議論は[こちら](https://github.com/NVIDIA/libnvidia-container/issues/148)（英語）

nVidiaの実験的リポジトリというところから`nvidia-container-toolkit=1.7.0~rc.1-1`を入手すれば良いらしい。

1. `curl`をインストール。

   ~~~shell
   $ sudo apt install curl
   ~~~

2. 以下を打つ。これで実験的リポジトリが`apt`のリポジトリリストに追加されるらしい。2つ目の`curl`の行は要らんかも。

   ~~~shell
   $ distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
      && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list \
      && curl -s -L https://nvidia.github.io/nvidia-container-runtime/experimental/$distribution/nvidia-container-runtime.list | sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
   ~~~

3. `apt`アップデート。

   ~~~shell
   $ sudo apt update
   ~~~

4. 入手できる`nvidia-container-toolkit`の確認。`1.7.0~rc.1-1`が見えていればOK。

   ~~~shell
   $ apt show nvidia-container-toolkit
   ~~~

   ~~~
   Package: nvidia-container-toolkit
   Version: 1.7.0~rc.1-1
   Priority: optional
   Section: utils
   Maintainer: NVIDIA CORPORATION <cudatools@nvidia.com>
   Installed-Size: 4,324 kB
   Depends: libnvidia-container-tools (>= 1.7.0~rc.1-1), libnvidia-container-tools (<< 2.0.0), libseccomp2
   Breaks: nvidia-container-runtime (<= 3.5.0-1), nvidia-container-runtime-hook
   Replaces: nvidia-container-runtime (<= 3.5.0-1), nvidia-container-runtime-hook
   Homepage: https://github.com/NVIDIA/nvidia-container-runtime/wiki
   Download-Size: 854 kB
   APT-Manual-Installed: yes
   APT-Sources: https://nvidia.github.io/libnvidia-container/experimental/ubuntu18.04/arm64  Packages
   Description: NVIDIA container runtime hook
     Provides a OCI hook to enable GPU support in containers.
   ~~~

5. `nvidia-container-toolkit`のインストール。

   ~~~shell
   $ sudo apt install nvidia-container-toolkit
   ~~~

これで動くようになる。

### 2. 古いバージョンを入れる

上記1.の解決策を避けたい時。

どのバージョンがインストールできるのかは以下でチェックできる。

~~~shell
$ apt-cache showpkg docker.io
~~~

~~~
Package: docker.io
Versions: 
20.10.7-0ubuntu5~18.04.3 (/var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic-updates_universe_binary-arm64_Packages) (/var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic-security_universe_binary-arm64_Packages) (/var/lib/dpkg/status)
 Description Language: 
                 File: /var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic_universe_binary-arm64_Packages
                  MD5: 05dc9eba68f3bf418e6a0cf29d555878
 Description Language: en
                 File: /var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic_universe_i18n_Translation-en
                  MD5: 05dc9eba68f3bf418e6a0cf29d555878
 Description Language: 
                 File: /var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic-updates_universe_binary-arm64_Packages
                  MD5: 05dc9eba68f3bf418e6a0cf29d555878

17.12.1-0ubuntu1 (/var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic_universe_binary-arm64_Packages)
 Description Language: 
                 File: /var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic_universe_binary-arm64_Packages
                  MD5: 05dc9eba68f3bf418e6a0cf29d555878
 Description Language: en
                 File: /var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic_universe_i18n_Translation-en
                  MD5: 05dc9eba68f3bf418e6a0cf29d555878
 Description Language: 
                 File: /var/lib/apt/lists/ports.ubuntu.com_ubuntu-ports_dists_bionic-updates_universe_binary-arm64_Packages
                  MD5: 05dc9eba68f3bf418e6a0cf29d555878


Reverse Depends: 
  nvidia-docker2,docker.io 18.06.0
  containerd,docker.io 19.03.6-0ubuntu1~18.04.3
  runc,docker.io 1.13.1~ds1-2
  containerd,docker.io 19.03.13-0ubuntu4
  cockpit-docker,docker.io 1.3.0
  vim-syntax-docker,docker.io
  runc,docker.io 1.13.1~ds1-2
  runc,docker.io 1.13.1~ds1-2
  whalebuilder,docker.io
  vim-syntax-docker,docker.io
  subuser,docker.io
  docker-compose,docker.io 1.9.0
  ruby-docker-api,docker.io
  packer,docker.io
  libnss-docker,docker.io
  htcondor,docker.io
  golang-libnetwork,docker.io 1.12
  gitlab-runner,docker.io
  docker-runc,docker.io 1.12
  docker-containerd,docker.io 1.12
  cockpit-docker,docker.io 1.3.0
  debocker,docker.io
  charliecloud,docker.io
Dependencies: 
20.10.7-0ubuntu5~18.04.3 - adduser (0 (null)) containerd (2 1.2.6-0ubuntu1~) iptables (0 (null)) debconf (18 0.5) debconf-2.0 (0 (null)) libc6 (2 2.17) libdevmapper1.02.1 (2 2:1.02.97) libsystemd0 (2 209~) docker (3 1.5~) ca-certificates (0 (null)) git (0 (null)) pigz (0 (null)) ubuntu-fan (0 (null)) xz-utils (0 (null)) apparmor (0 (null)) aufs-tools (0 (null)) btrfs-progs (0 (null)) cgroupfs-mount (16 (null)) cgroup-lite (0 (null)) debootstrap (0 (null)) docker-doc (0 (null)) rinse (0 (null)) zfs-fuse (16 (null)) zfsutils (0 (null)) docker (3 1.5~) 
17.12.1-0ubuntu1 - adduser (0 (null)) iptables (0 (null)) debconf (18 0.5) debconf-2.0 (0 (null)) libc6 (2 2.17) libdevmapper1.02.1 (2 2:1.02.97) libltdl7 (2 2.4.6) libseccomp2 (2 2.3.0) docker-containerd (0 (null)) docker-runc (0 (null)) docker (3 1.5~) ca-certificates (0 (null)) cgroupfs-mount (16 (null)) cgroup-lite (0 (null)) git (0 (null)) ubuntu-fan (0 (null)) xz-utils (0 (null)) apparmor (0 (null)) aufs-tools (0 (null)) btrfs-tools (0 (null)) debootstrap (0 (null)) docker-doc (0 (null)) rinse (0 (null)) zfs-fuse (16 (null)) zfsutils (0 (null)) docker (3 1.5~) docker-containerd (0 (null)) docker-runc (0 (null)) 
Provides: 
20.10.7-0ubuntu5~18.04.3 - 
17.12.1-0ubuntu1 - docker-runc (= ) docker-containerd (= ) 
Reverse Provides: 

~~~

このことからインストール可能なバージョンは

* `20.10.7-0ubuntu5~18.04.3`
* `17.12.1-0ubuntu1`

ということが分かるので、以下を打って古いバージョンをインストールし直す。

~~~shell
$ sudo apt install docker.io=17.12.1-0ubuntu1
~~~

確認。

~~~shell
$ docker -v
Docker version 17.12.1-ce, build 7390fc6
~~~

### バージョンを固定する

~~~shell
$ sudo apt-mark hold docker.io
~~~

将来的に問題が解消されたら固定を解除すれば良い。

~~~shell
$ sudo apt-mark unhold docker.io
~~~

ちなみにバージョンが固定されているパッケージを全部見るには以下。

~~~shell
$ apt-mark showhold
~~~

参考：[hanhan's blog - Ubuntuで古いバージョンのパッケージを使う](https://blog.hanhans.net/2021/02/17/ubuntu-apt-old-package/)


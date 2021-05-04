# 用意したPCとUbuntuインストール概略

## 用意したPC

[Product – Minisforum](https://store.minisforum.com/pages/product)　「N40」

| 項目                  | 内容                                                         |
| --------------------- | ------------------------------------------------------------ |
| Processor             | Intel® Celeron® Processor N4000 , 2 Cores/2 Threads (4M Cache, up to 2.60 GHz) |
| GPU                   | Intel® UHD Graphics 600                                      |
| Memory                | LPDDR4 4GB (On Board)                                        |
| Storage               | eMMC 64GB (On Board)                                         |
| Storage Expansion     | 1×M.2 2242 SATA SSD Slot (up to 1TB , SATA 3.0 6.0Gb/s) 1×SD Card Slot (up to 128GB) |
| Wireless Connectivity | IEEE 802.11ac Dual-Band Wi-Fi，BT4.2                         |
| Ethernet              | 1000Mbps LAN                                                 |
| Video Output          | ① HDMI 2.0(4K@60Hz), ② VGA Type Port                         |
| Audio Output          | HDMI 2.0 , 3.5mm Audio Jack                                  |
| Peripherals Interface | RJ45 Gigabit Ethernet Port×1，USB 3.0 Port×3 , SD Card Slot×1 , DC Jack×1 |
| Power                 | DC 12V/1.5A (adapter included)                               |
| System                | Windows 10 Pro                                               |
| Launch Date           | December-19                                                  |

Youtubeで動画を見てこれだと思って購入。Amazonで2万円。

ポイントは

* Celeron→省電力。
* ファンレス→うるさくない。
* USB3.0→Raspberry Piだと3.0があるのは4から。でも4にはファンが必要。
* サイズ→他にありそうであまり無い。

## インストール概略

大体はRaspberry PiとかGaruda Linuxと雰囲気は一緒なので概略だけ。

1. [公式HP](https://ubuntu.com/download/server)からISOイメージをゲット
2. [Rufus](https://rufus.ie/ja/)でISOイメージをUSBに書き込み
3. PCのBIOSに入り、セキュアブートOFFにしてUSBからのブートを最優先
4. ISOイメージを書き込んだUSBを差してPC立ち上げ
5. 説明に従ってインストールを進める
6. 指示どおりにUSBを抜き、再起動
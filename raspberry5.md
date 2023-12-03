# 4年ぶりの進化、Raspberry Pi 5が発表されたのでその興奮をお届けします

「いやー、この前Raspberry Pi 4買ったんすよ。めっちゃおもろいね。昨日ラズパイネタで登壇しちゃったよ」

「今日、Raspberry Pi 5、発表されたってよ？」

「（この業界怖いわ…）」

というわけで、先日の2023年9月27日（水）に、下のイベントでハマコー登壇しました。

[【9/27（水）東京】クラスメソッドの最新開発ノウハウを学ぶ勉強会 in 東京 〜IoT最前線〜 \- connpass](https://classmethod.connpass.com/event/296208/)

ドヤァとRaspberry Pi 4の実物を見せながら意気揚々と喋ったわけですが、その翌日に、Raspberry Pi 5発表のニュースが飛び込んできました。なんなんすかね、このめぐり合わせ。

[Introducing: Raspberry Pi 5\! \- Raspberry Pi](https://www.raspberrypi.com/news/introducing-raspberry-pi-5/)

ちょうど、ラズパイを触り始めた直後ということもあり、4からの進化に興奮する点が多々あったので、皆さんにその興奮をお届けできればと思ってます！

<pre style="line-height:120%;">
Pi 5きたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## 公式情報

Raspberry Piは、イギリスの[Raspberry Pi Foundation](https://www.raspberrypi.org/)が製造元のシンブルボードコンピュータ。最新バージョンが4だったところに、このたびめでたく、5が発表されたというわけです。

[Introducing: Raspberry Pi 5\! \- Raspberry Pi](https://www.raspberrypi.com/news/introducing-raspberry-pi-5/)

1から4の進化を簡単にまとめた表がこちら。2012年に初代の1が発表され、最新バージョンの4が2019年に発表されていました。今回は4年ぶりに5が発表されました。ビッグニュースですね！

| 仕様/モデル     | Raspberry Pi 1            | Raspberry Pi 2         | Raspberry Pi 3                     | Raspberry Pi 4                 |
|------------|-------------------------|----------------------|--------------------------------|--------------------------|
| リリース年   | 2012                     | 2015                 | 2016                           | 2019                     |
| CPU        | Single-core ARM1176JZF-S | Quad-core Cortex-A7  | Quad-core Cortex-A53 (64-bit)  | Quad-core Cortex-A72    |
| 速度        | 700 MHz                  | 900 MHz              | 1.2 GHz                        | 1.5 GHz                  |
| RAM        | 256MB/512MB              | 1GB                  | 1GB                            | 1GB/2GB/4GB/8GB         |
| 通信        | Ethernet                 | Ethernet             | Wi-Fi, Bluetooth, Ethernet     | Wi-Fi, Bluetooth, Gigabit Ethernet |
| USBポート  | 2                        | 4                    | 4                              | 2×USB3.0, 2×USB2.0      |
| 映像出力    | HDMI, RCA                | HDMI, RCA            | HDMI, 3.5mm jack               | 2×micro-HDMI           |
| 電源        | Micro USB                | Micro USB            | Micro USB                      | USB-C                   |
| その他      | 40-pin GPIO              | 40-pin GPIO          | 40-pin GPIO, CSI, DSI          | 40-pin GPIO, CSI, DSI, USB-C for Power |

以下、進化の様子をかいつまんでお届けいたします。


## リリース日と値段

リリース日は2023年10月末予定。価格はメモリ4GBが$60、8GBが$80とのこと。もちろん日本で入手するときは為替や送料がかかるので、そのまんまこの値段ってわけにはならないでしょうが、今、Raspberry Pi 4のメモリ8GBを日本で入手しようとすると、だいたい1万2〜4千円ぐらいすることを考えると、最新バージョンなのに安く感じますね。

## 主要スペック（4との比較）

ざーっと、主要スペックについて4との比較をまとめてみました。こちら眺めてもらいながら、進化点解説していきます。

| 仕様/モデル     | Raspberry Pi 4                      | Raspberry Pi 5                                           |
|------------|----------------------------------|-------------------------------------------------------|
| CPU        | Quad-core Cortex-A72 (1.5 GHz)   | 2.4GHz quad-core 64-bit Arm Cortex-A76               |
| GPU        | VideoCore VI                     | VideoCore VII (OpenGL ES 3.1, Vulkan 1.2対応)          |
| ディスプレイ出力 | 2×micro-HDMI                  | Dual 4Kp60 HDMI®                                       |
| デコーダ    | -                               | 4Kp60 HEVC                                             |
| 通信        | Wi-Fi, Bluetooth, Gigabit Ethernet | Dual-band 802.11ac Wi-Fi®, Bluetooth 5.0 / BLE        |
| USBポート  | 2×USB3.0, 2×USB2.0              | 2 × USB 3.0 ports (5Gbps), 2 × USB 2.0 ports          |
| その他      | 40-pin GPIO, CSI, DSI, USB-C for Power | 40-pin GPIO, 2×4-lane MIPI, PCIe 2.0 x1, Real-time clock, Power button |

## 外観の変化（4との比較）

最近買ったPi 4のボード写真。

010

公式から拝借した、Pi 5のボード写真。

020

わかりやすいところからいくと、右側、4は、上からイーサネットポート、USB 3.0✕2、USB 2.0✕2の順番になっているのが、5は、上からUSB 2.0✕2、USB 3.0✕2、イーサネットポートと、逆順になっているところです。

奥側のGPIO40ピンの場所は、ほぼ変わってなさそうです。手前側のマイクロHDMI✕2と、給電用のUSB Type-Cは、場所はほぼ変わらず。その右側、以前はあった4極のビデオオーディオジャックは削除され、あたらしくFPCコネクタが追加されています。

4極ジャックとカメラコネクタがあった場所は、新しく2つの










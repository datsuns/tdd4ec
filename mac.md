# STM32F4 Discovery環境構築 for Mac

## 参考サイト

* http://www51.atwiki.jp/kougyou_kei/pages/17.html
   * win向け手順が書かれている
* http://www.radiohobbyist.org/blog/?p=1337
   * アーキテクチャ図とかもあっていいかも
   * https://github.com/ehntoo/summon-arm-toolchain
   * openocdで指定しているファイルは以下にある
      * <summon-installed>/sat/share/openocd/scripts/board
* https://github.com/esden/summon-arm-toolchain.git
   * 結局ココみたい
* http://projectc3.seesaa.net/article/283806921.html
   * サンプルコードはここから持ってきたりもした
* http://strobo.hatenablog.com/entry/2013/06/30/155220
   * openocdのコマンドとか詳細を書いてくれてる。おすすめ
* https://github.com/ThrowTheSwitch/Unity
   * Unityはgithubに移行したみたい

## STM32F4データシート

* http://pdf1.alldatasheet.jp/datasheet-pdf/view/510597/STMICROELECTRONICS/STM32F407IE.html

## ツールのビルド手順

*失敗含めて記載しているので先に手順をひと通り見てもらった方がいいかも*

1. mkdir as/you/like/[work-dir] && cd [work-dir]
1. git clone https://github.com/esden/summon-arm-toolchain.git
1. READMEにしたがってもろもろインストール
   * mpfr gmp libmpc texinfo libusb-compat libftdi wget
   * wgetのインストールに手間取ったけどportのselfupdateやったらwgetも更新されて普通に動くようになった
1. cd summon-arm-toolchain
   * このツールはいシステムへのインストールまではやらないみたい（ビルド済バイナリを集めてくれるところまで）
   * バイナリの出力先は PREFIX で指定
      * **PREFIX は絶対パスでないとダメぽい**
1. ./summon-arm-toolchain PREFIX=/Users/datsuns/bin/sat CPUS=4
   * *gccのコンパイルでコケる*
   * DARWIN_OPT_PATHを修正する（かsummon-arm-toolchainの起動オプションで指定する）
      * - DARWIN_OPT_PATH=/usr/local	# Path in which MacPorts or Fink is installed
      * + DARWIN_OPT_PATH=/opt/local	# Path in which MacPorts or Fink is installed
   * /opt/localはMacPorts。HomeBrewは別のパスかも（未確認
   * OpenOCDはJTAG関連みたいなのでOFF ?? (結局ONでビルドした)
1. 再度: ./summon-arm-toolchain CPUS=4
   * OpenOCDがなんとかいうのがコンパイルエラーに成る
      * libftdiがなんとかかんとか
         * port install libftdi でエラー
         * **libftdiではなくてlibftdi0をインストールする**
            * と、MacPortsのエラーログにあったのでそのとおりに
1. ./summon-arm-toolchain CPUS=4 OOCD_EN=1 PREFIX=/Users/datsuns/bin/sat
   * libopencm3でエラー。pythonのyamlロードでこけてる。py27-yaml入れたけど？
   * 手動で入れる
      * wget http://pyyaml.org/download/pyyaml/PyYAML-3.10.tar.gz
      * tar -xf
      * sudo python setup.py install
1. 改めて実行 ./summon-arm-toolchain CPUS=4 OOCD_EN=1 PREFIX=/Users/datsuns/bin/sat
   * **行けたくさい**
1. どうもstm32用のライブラリも有効にしないとだめくさい
   * ./summon-arm-toolchain CPUS=4 OOCD_EN=1 LIBSTM32_EN=1 PREFIX=/Users/datsuns/bin/sat
   * **これはすんなりいけたくさい**

## 実環境用バイナリのビルド方法

* この辺りなんかを参考に
   * http://strobo.hatenablog.com/entry/2013/06/30/155220
   * 結局は「サンプル落としてきてビルド」が手っ取り早い
   * summon-arm-toolchainでのビルドは出力バイナリのパスを$PATHに入れてくれないのでmakeする前にパスは通しておく
      * こんなかんじで
         * PATH=/$PATH:~/bin/sat/bin
         * システム全体に効かせるのが嫌だったので上のを記載したファイルを作ってsourceするだけにとどめておく

## 実機との接続

* http://strobo.hatenablog.com/entry/2013/06/30/155220
   * openocdのコマンドとかひと通り書いてくれるのでわかりやすい
      * openocdのオプションはsummon-arm-toolchainのPREFIXに指定したパスを考慮する
         * PREFIX=~/bin/satでのopenocd起動例
            1. cd ~/bin/sat
            1. ./bin/openocd -s ./share/openocd/scripts -f ./share/openocd/scripts/interface/stlink-v2.cfg -f ./share/openocd/scripts/target/stm32f4x_stlink.cfg
   * サンプルの挙動
      * ボタンの間のLEDが交互に光る
   * *ちょっと注意*
      * openocdはデフォルトでポート3333を使う模様
          * openocdの起動オプションとかで変更できるとは思うが今回はそのままで。
      * loadした後に一旦接続が切れる？ので**targetコマンドが合計２回必要な場合がある**
      * 以下参考用のgdb側の実行ログ

      > (gdb) target remote localhost:3333   ★ １回目のtarget

      > Remote debugging using localhost:3333

      > 0x00000000 in ?? ()

      > (gdb) monitor reset halt

      > target state: halted

      > target halted due to debug-request, current mode: Thread

      > xPSR: 0x01000000 pc: 0x08000ddc msp: 0x10010000

      > (gdb) load

      > Loading section startup, size 0x188 lma 0x8000000

      > Loading section .text, size 0x7dd8 lma 0x8000190

      > Loading section .data, size 0xec lma 0x8007f68

      > Start address 0x80002e1, load size 32844

      > Remote connection closed            ★ なぜか一旦切れる？

      > (gdb) target remote localhost:3333  ★ ２回目のtarget

      > Remote debugging using localhost:3333

      > ResetHandler () at ../../os/ports/GCC/ARMCMx/crt0.c:274

      > 274	  asm volatile ("cpsid   i");

      > (gdb) monitor reset halt

      > target state: halted

      > target halted due to debug-request, current mode: Thread

      > xPSR: 0x01000000 pc: 0x080002e0 msp: 0x20000400

      > (gdb) continue

      > Continuing.


## USB-Serialでの動作デモ

まずはprintf()ライクな関数が動くよね？を確かめようかと。今回のサンプルはChibiOSに依存しているため、そのAPI **chprintf()** というAPIでログ出力している模様。

* 参考
   * http://forum.chibios.org/phpbb/viewtopic.php?f=4&t=514&start=10

1. 事前に何かしらのシリアルターミナルをインストール
   * macにはscreenが標準で入ってる？のでそれも使える
   * ワタシは昔Arduino(ino)用に入れてたpicocomというのを利用
1. ChibiOS本体をダウンロード
   * https://github.com/ChibiOS/ChibiOS-RT
   * とりあえずmasterでもサンプルは動いたけどどこかのtagにでもcheckoutしておいたほうがいいかも？
      * と思ったらmasterしか無いのでそのままでいいでしょう
1. サンプルのダウンロード
   * https://github.com/resset/STM32F4-Discovery-example-code.git
1. makefile中のChibiOSへのパスをダウンロードしたChibiOSへのパスで修正
   * @STM32F4-Discovery-example-code/makefile
      * CHIBIOS = ../ChibiOS-RT/
1. make
   * とおるはず！
1. 実機の接続
   1. USB mini / micro 双方をPCに
      * このサンプルだとmicro側を出力に使ってるみたい
1. 後はこれまでと同様にopenocdにgdbでremote接続してボードにプログラムをloadし、go
1. PCのシェルからls /dev として自動認識されるはずのCDCデバイスを確認
   * 試した環境[Max OS X 10..7.5]だと別途ドライバ等不要でした
   * 試した時の状態だとこんな結果

   > % ls /dev | grep usb

   >  cu.usbmodem271

   >  tty.usbmodem271
1. シリアルターミナル起動！
   * 見つかったttyデバイスファイルを指定してターミナルを起動
      * picocom /dev/tty.usbmodem271
   * サンプルだと「ch>」なプロンプトが出るのでそれでOK!
      * help でコマンド一覧とかが出てくる

## CppUTestのビルド

1. configure実行
> CFLAGS=-mcpu=cortex-m4 -nostartfiles -T/Users/datsuns/work/programming/cpp/tdd4ec/stm32fdiscovery/ChibiOS/ChibiOS_2.6.1/os/ports/GCC/ARMCMx/STM32F4xx/ld/STM32F407xG.ld -Wl,-Map=build/ch.map,--cref,--no-warn-mismatch,--gc-sections  -mno-thumb-interwork -mthumb ./configure --host=arm-none-eabi --disable-std-c

   1. オプションはChibiOSのdemoをvervoseビルドして割り出した


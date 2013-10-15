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

* projectc3.seesaa.net/article/283806921.html
   * サンプルコードはここから持ってきた

* http://strobo.hatenablog.com/entry/2013/06/30/155220
   * openocdのコマンドとか詳細を書いてくれてる。おすすめ

## STM32F4データシート

* http://pdf1.alldatasheet.jp/datasheet-pdf/view/510597/STMICROELECTRONICS/STM32F407IE.html

## ツールのビルド手順

1. cd <work-dir>
1. git clone https://github.com/esden/summon-arm-toolchain.git
1. READMEにしたがってもろもろインストール
   * mpfr gmp libmpc texinfo libusb-compat libftdi wget
   * wgetのインストールに手間取ったけどportのselfupdateやったらwgetも更新されて普通に動くようになった
1. cd summon-arm-toolchain
   * インストールディレクトリは PREFIX で指定
      * PREFIX は絶対パスでないとダメぽい
1. ./summon-arm-toolchain PREFIX=/Users/datsuns/bin/sat CPUS=4
   * gccのコンパイルでコケる
   * DARWIN_OPT_PATHを修正する（か指定する）
      * - DARWIN_OPT_PATH=/usr/local	# Path in which MacPorts or Fink is installed
      * + DARWIN_OPT_PATH=/opt/local	# Path in which MacPorts or Fink is installed
   * OpenOCDはJTAG関連みたいなのでOFFにする
1. 再度: ./summon-arm-toolchain CPUS=4
   * OpenOCDがなんとかいうのがコンパイルエラーに成る
      * libftdiがなんとかかんとか
         * port install libftdi でエラー
         * **libftdiではなくてlibftdi0をインストールする**
   * OpenOCDはJTAG関連みたいなのでOFFにする
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

## 実機との接続

* http://strobo.hatenablog.com/entry/2013/06/30/155220
   * openocdのコマンドとかひと通り書いてくれる


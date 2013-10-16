# 環境構築 linux編

## ツールチェインのインストール

1. Sourcery CodeBench Lite Editionを使う
   * http://www.mentor.com/embedded-software/sourcery-tools/sourcery-codebench/editions/lite-edition/
1. 「ARM Processors」の「Download the EABI Release」をクリック
   * 氏名、Emailアドレスなど入れる
1. メールで送られてきたURLにアクセスして「Sourcery CodeBench Lite 2013.05-23」の「IA32 GNU/Linux TAR」を展開
1. 展開したパスのbin以下にパスを通す

* 参考URL)
   * [STM32F4 DiscoveryをUbuntuから試食してみました](http://projectc3.seesaa.net/article/283806921.html)

## libusbのインストール

1. apt-getなりからインストールしておきましょう

## サンプルプログラムのダウンロード
1. 参考URLからSTM32F4_Sample_forLinuxBuildEnv.tgzをダウンロード、展開、makeする
1. main.elfができることを確認

* 参考URL)
   * [STM32F4 DiscoveryをUbuntuから試食してみました](http://projectc3.seesaa.net/article/283806921.html)

## OpenOcdのインストール

1. 下記からopenocd-0.7.0.tar.bz2 をダウンロード、展開
   * http://sourceforge.net/projects/openocd/files/openocd/
1. 下記の手順でビルド、インストール
   1. ./configure --enable-maintainer-mode --enable-stlink
   1. make
   1. sudo make install
1. /usr/local/bin、/usr/local/shareの下にopenocdディレクトリができます

* 参考URL)
   * [STM32F4-discoveryデバッグ環境にOpenOCDをセットアップ](http://projectc3.seesaa.net/article/285823341.html)
   * [MacでSTM32F4-Discoveryの開発環境を構築してChibiOS/RTを動かす](http://strobo.hatenablog.com/entry/2013/06/30/155220)


## 基板への書き込み
**プリインされているサンプルは、上書きされますので念のため注意**

1. mini-BのUSBケーブルでPCと接続
1. OpenOCDを起動
   * openocd -s tcl -f /usr/local/share/openocd/scripts/board/stm32f4discovery.cfg
1. OpenOCDを起動すると窓が専有されるので別窓から下記のコマンドでgdbを起動
   * arm-none-eabi-gdb main.elf
   * main.elfは、ビルドしてできたファイル
1. gdbで下記を実行して書き込み
   1. target remote localhost:3333
   1. monitor reset halt
   1. load
   1. monitor reset halt
1. 実行
   1. continue

### サンプルプログラムの挙動
* User、Resetボタンの間にある４つのLEDがチカチカしました

* 参考URL)
   * [MacでSTM32F4-Discoveryの開発環境を構築してChibiOS/RTを動かす](http://strobo.hatenablog.com/entry/2013/06/30/155220)
   * [STM32VLDiscovoryでOpenOCDを使うメモ](http://www.shortplug.jp/profile/75/diaries/2940)


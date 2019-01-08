# modulo-bt-pendant

## ビルドガイド
D1, Q1, R1, R2をはんだ付けします。D1のみ極性に注意してください。

D4--D7はオプションです。外部から5V給電を受け付ける場合は取り付けてください。
スイッチOFF時のみ外部給電モードに入ります。

電池ボックス、TRRSプラグは必要に応じて表面または裏面に取り付けます。
2個のBT PendantをErgo42 moduloと組み合わせる場合、片方のモジュールではこれらを裏面に取り付けると対称に装着できます。
Ergo42 Moduloでは4つのTRRSジャックのうち外側の2つがI2C用です。BT PendantのプラグのI2Cと書いてある側を外側に差し込んでください。

SWDまたはブートローダから書き込みます。SWDから書き込む場合、ブートローダ・ソフトデバイスを一度上書きする必要があります。

## ブートローダを使った書き込み・動作確認
Ergo42 Moduloとの組み合わせを想定した動作確認の方法を説明します。

左手側がマスタ、右手側がスレーブの完全無線構成向けです。
また、Ergo42 Moduloのデバイスアドレスは0x26, 0x27を想定しています。(DIPスイッチが両方ともOFF)



### ファームウェアの書き込み

1. スマートフォン・タブレットに[nRF Toolbox](https://play.google.com/store/apps/details?id=no.nordicsemi.android.nrftoolbox&hl=ja)をインストール(現時点ではPC版からは書き込めないようです。)

1. [Release](https://github.com/sekigon-gonnoc/modulo-bt-pendant/releases)からデモプログラムをダウンロード

1. BT Pendantのデバッグ用ポート(V,S,C,/R,G,T,R)のうち、GとRをショートさせながら電源を投入(ブートローダが起動します。初回は不要です。)

1. nRF ToolboxでDFUを選択し起動

1. SELECT FILEから先ほどダウンロードしたzipファイルを選択

1. SELECT DEVICEで**DFU Targ**を選択

1. UPLOADを押し、書き込みを開始

### 動作確認

1. スレーブ側のみを起動し、スマートフォン等からBluetoothデバイスのスキャンを実行

1. デバイス一覧に**Nordic UART**という表示があることを確認したら、マスタ側を起動

1. デバイス一覧から**Nordic UART**という表示が消え、代わりに**ModuloBLE**が表示されることを確認後、ペアリングを実行

以上の手順により、無線キーボードとしての動作確認ができます。
これ以降、スレーブ側のみを起動したとしても**Nordic UART**はデバイス一覧に表示されません。


## ビルド環境構築

基本的には[BLE Micro Pro](https://github.com/sekigon-gonnoc/BLE-Micro-Pro)
のものを参照してください。
ただし、SDKは[v12.3.0](https://www.nordicsemi.com/Software-and-Tools/Software/nRF5-SDK/Download#infotabs)をダウンロードし(技適の都合)、環境変数の名前は**NRFSDK12_ROOT**としてください。

## hexファイルから書き込み用パッケージの生成
1. [nrfutil.exe](https://github.com/NordicSemiconductor/pc-nrfutil/releases)をダウンロードあるいはpipからインストール(python2.7)

1. [Release](https://github.com/sekigon-gonnoc/modulo-bt-pendant/releases)から`modulo_bt_pendant.pem`をダウンロード

1. nrfutilを使ってパッケージを生成

```
nrfutil pkg generate --application FIRMWARE_NAME.hex --debug-mode --hw-version 0 --sd-req 0x8c --key-file modulo_bt_pendant.pem FIRMWARE_NAME.zip
```

## SWDを使った書き込み

CMSIS-DAPとOpenOCD(0.10.0)を使った書き込み方法を説明します。OpenOCDはST-Link等にも対応しているはずですが未確認です。

### 書き込み機との接続
デバッグ用ポート(V,S,C,/R,G,T,R)のSをSWDIO,　CをSWCLK, GをGNDに接続します。

### 初期化とソフトデバイスの書き込み
この手順は最初に一度実行すれば以降は不要です。
```
openocd -f interface/cmsis-dap.cfg -f target/nrf52.cfg -c init -c "reset init" -c halt -c "nrf5 mass_erase" -c "program s132_nrf52_3.0.0_softdevice.hex verify" -c reset -c exit
```

### ファームウェアの書き込み

```
openocd -f interface/cmsis-dap.cfg -f target/nrf52.cfg -c "program FIRMWARE_NAME.hex verify" -c "reset" -c exit
```


# IoT水耕栽培をはじめてみる
## 目次
1. はじめに
2. キット内容物確認
3. セットアップ手順
   - Raspberry Pi Zero WH OSインストール
   - Raspberry Pi Zero WH SSH、Wi-Fi設定
   - Grove Base HAT、センサ取り付け
   - Ambientチャネル作成
   - 温湿度センサ用プログラム作成
   - TDS水質センサ用プログラム作成
   - プログラム自動起動設定
   - Ambient動作確認
   - 水耕栽培キット組み立て
   - Raspberry Pi/水耕栽培キットの連結


## 1. はじめに
マンションのベランダで家庭菜園をやっているが、
- 東向きのため午前中は日が入るがお昼以降は日が入らない
- 日当たりの悪さか、他に場所がないのか、コバエの住処にされて困る

そこで太陽の日差しがなくても部屋の中で育てられる水耕栽培をはじめてみる。
せっかくなので、ラズパイでセンサのデータを取得して植物の生育と環境情報を見える化する。

## 2. キット内容物確認

<img width="500" src="https://github.com/takamasa-s/iot-hydroponics/blob/main/kit.png">

- [ ] ①Raspberry Pi Zero WH

- [ ] ②Raspberry Pi用Grove Base Hat

- [ ] ③Grove - TDS水質センサ

- [ ] ④Grove - 温度及び湿度センサ

- [ ] ⑤Grove 4-pin（予備）

- [ ] ⑥USBケーブル

- [ ] ⑦SDカード

- [ ] 水耕栽培キット

https://www.motom-jp.com/2021/03/29/oma14/

## 3. セットアップ手順

### Raspberry Pi Zero WH OSインストール

①イメージャーのダウンロード

https://www.raspberrypi.org/software/

②Raspberry Pi OS (32-bit)を選択してインストール

### Raspberry Pi Zero WH SSH、Wi-Fi設定

①コマンドプロンプトから下記を実行。
```shell
type nul > D:\ssh
type nul > D:\wpa_supplicant.conf
```
*Dドライブの場合

②wpa_supplicant.confに下記を記載。SSIDとパスワードは自身のものを。
```
country=JP
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
   ssid="SSID"
   psk="password"
}
```

③Bonjourインストール

https://download.info.apple.com/Mac_OS_X/061-8098.20100603.gthyu/BonjourPSSetup.exe

インストール後、PC再起動

④Raspberry PiにMicro SBカードをさしてUSBケーブルで給電
（緑のLEDが点灯したらOS起動完了）

⑤Teraterm等端末エミュレータで接続
ホスト名：raspberripi.local
ユーザ名：pi
パスワード：raspberry

⑥パッケージ更新
```bash
pi@raspberrypi:~ $ sudo apt-get update -y
pi@raspberrypi:~ $ sudo apt-get upgrade -y
pi@raspberrypi:~ $ sudo apt-get dist-upgrade -y
```
### Grove Base HAT、センサ取り付け

①Raspberry Piの電源を切る（USBから抜く）

②Raspberry Piに支柱を取り付ける

③Base HATをRaspberry Piに取り付ける

④温度及び湿度センサをBase HATのPWMポートに、TDS水質センサをA0ポートに接続

### Ambientチャネル作成

①下記サイトにてアカウント作成

https://ambidata.io/

②`チャネル一覧`から`チャネルを作る`をクリック

### 温湿度センサ用プログラム作成


### TDS水質センサ用プログラム作成


### プログラム自動起動設定



### Ambient動作確認


### 水耕栽培キット組み立て

説明書の通りに組み立てる

### Raspberry Pi/水耕栽培キットの連結

TDS水質センサを水耕栽培キットの空いている穴に差してAmbientでデータが取れていることを確認

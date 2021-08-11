# IoT水耕栽培、はじめてみた
## 目次
1. はじめに
2. キット内容物確認
3. セットアップ手順
   - Raspberry Pi Zero WH OSインストール
   - Raspberry Pi Zero WH SSH、Wi-Fi設定
   - Grove Base HAT、センサ取り付け
   - Ambientチャネル作成
   - 温湿度センサ用クラス作成
   - TDS水質センサ用クラス作成
   - 実行プログラム作成
   - プログラム自動起動設定
   - Ambient動作確認
   - 水耕栽培キット組み立て
   - Raspberry Pi/水耕栽培キットの連結
4. 参考にさせていただいたサイト

<br />

## 1. はじめに
マンションのベランダで家庭菜園をやっているのですが、
- 東向きのため午前中は日が入るがお昼以降は日が入らない
- 日当たりの悪さか、他に住処がないのか、キノコバエの家にされて困る

そこで太陽の日差しがなくても部屋の中で育てられる水耕栽培をはじめてみました。

せっかくなので、ラズパイでセンサのデータを取得して植物の生育と環境情報を見える化したいと思います。

<br />

キットを組み立て終わると、こんな完成系になりました。

<img width="500" src="https://github.com/takamasa-s/iot-hydroponics/blob/main/iot_suiko.jpg">

<img width="700" src="https://github.com/takamasa-s/iot-hydroponics/blob/main/ambient.png">

## 2. キット内容物確認

<img width="500" src="https://github.com/takamasa-s/iot-hydroponics/blob/main/kit.png">

- [ ] ① Raspberry Pi Zero WH

- [ ] ② Raspberry Pi用Grove Base Hat

- [ ] ③ Grove - TDS水質センサ

- [ ] ④ Grove - 温度及び湿度センサ

- [ ] ⑤ Grove 4-pin（予備）

- [ ] ⑥ マイクロUSBケーブル

- [ ] ⑦ SDカード（Raspberry Pi OSの要求により4GB以上（8GB以上推奨））

- [ ] 水耕栽培キット [LED水耕栽培器 Akarina14 (OMA14)](https://www.motom-jp.com/2021/03/29/oma14/)

<br />

## 3. セットアップ手順

### Raspberry Pi Zero WH OSインストール

① イメージャーのダウンロード

https://www.raspberrypi.org/software/

② Raspberry Pi OS (32-bit)を選択してインストール

<br />

### Raspberry Pi Zero WH SSH、Wi-Fi設定

① コマンドプロンプトから下記を実行。
```shell
type nul > D:\ssh
type nul > D:\wpa_supplicant.conf
```
*Dドライブの場合

② wpa_supplicant.confに下記を記載。SSIDとパスワードは自身のものを。
```
country=JP
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
   ssid="SSID"
   psk="password"
}
```

③ Bonjourインストール

https://download.info.apple.com/Mac_OS_X/061-8098.20100603.gthyu/BonjourPSSetup.exe

インストール後、PC再起動

④ Raspberry PiにMicro SBカードをさしてUSBケーブルで給電
（緑のLEDが点灯したらOS起動完了）

⑤ Teraterm等端末エミュレータで接続

ホスト名：`raspberripi.local`

ユーザ名：`pi`

パスワード：`raspberry`

⑥ パッケージ更新
```bash
pi@raspberrypi:~ $ sudo apt-get update -y
pi@raspberrypi:~ $ sudo apt-get upgrade -y
pi@raspberrypi:~ $ sudo apt-get dist-upgrade -y
```

⑦ タイムゾーン変更
```bash
pi@raspberrypi:~ $ timedatectl
pi@raspberrypi:~ $ timedatectl list-timezones | grep Asia/Tokyo
pi@raspberrypi:~ $ sudo timedatectl set-timezone Asia/Tokyo
pi@raspberrypi:~ $ timedatectl
```

⑧ grove.pyインストール
```bash
pi@raspberrypi:~ $ curl -sL https://github.com/Seeed-Studio/grove.py/raw/master/install.sh | sudo bash -s -
```

⑨ AmbientのPython用ライブラリ（ambient-python-lib）インストール
```bash
pi@raspberrypi:~ $ sudo pip3 install git+https://github.com/AmbientDataInc/ambient-python-lib.git
pi@raspberrypi:~ $ sudo pip3 install schedule
```

⑩ I2C（Inter Integrated Circuit）有効化
```bash
pi@raspberrypi:~ $ sudo apt-get install i2c-tools
pi@raspberrypi:~ $ sudo raspi-config
```
`3 Interface Options`⇒`P5 I2C`⇒`Yes`⇒`OK`を選択。
```bash
pi@raspberrypi:~ $ i2cdetect -y 1
```


⑪ vim-gtkインストール（vimのヤンクをクリップボード経由でおこなえるようにする）
```bash
pi@raspberrypi:~ $ sudo apt remove -y vim
pi@raspberrypi:~ $ sudo apt-get install -y vim-gtk
pi@raspberrypi:~ $ echo "set clipboard=unnamedplus" >> .vimrc
```

<br />

### Grove Base HAT、センサ取り付け

① Raspberry Piの電源を切る（USBから抜く）

② Raspberry Piに支柱を取り付ける

③ Base HATをRaspberry Piに取り付ける

④ 温度及び湿度センサをBase HatのPWMポートに、TDS水質センサをA0ポートに接続

[参考] [Raspberry Pi用Grove Base Hat Wiki](https://wiki.seeedstudio.com/Grove_Base_Hat_for_Raspberry_Pi/)

<br />

### Ambientチャネル作成

① 下記サイトにてアカウント作成

https://ambidata.io/

② `チャネル一覧`から`チャネルを作る`をクリック

<br />

### 温湿度センサ用クラス作成

`/home/pi/sensor/`に`grove_temperature_humidity_sensor.py`を作成。

<details>
<summary>温湿度センサ用クラス</summary>

```python:grove_temperature_humidity_sensor.py
import RPi.GPIO as GPIO
# from grove.helper import *
def set_max_priority(): pass
def set_default_priority(): pass
from time import sleep

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

PULSES_CNT = 41

class DHT(object):
    DHT_TYPE = {
        'DHT11': '11',
        'DHT22': '22'
    }

    MAX_CNT = 320

    def __init__(self, dht_type, pin):        
        self.pin = pin
        if dht_type != self.DHT_TYPE['DHT11'] and dht_type != self.DHT_TYPE['DHT22']:
            print('ERROR: Please use 11|22 as dht type.')
            exit(1)
        self._dht_type = '11'
        self.dht_type = dht_type
        GPIO.setup(self.pin, GPIO.OUT)

    @property
    def dht_type(self):
        return self._dht_type

    @dht_type.setter
    def dht_type(self, type):
        self._dht_type = type
        self._last_temp = 0.0
        self._last_humi = 0.0

    def _read(self):
        # Send Falling signal to trigger sensor output data
        # Wait for 20ms to collect 42 bytes data
        GPIO.setup(self.pin, GPIO.OUT)
        set_max_priority()

        GPIO.output(self.pin, 1)
        sleep(.2)

        GPIO.output(self.pin, 0)
        sleep(.018)

        GPIO.setup(self.pin, GPIO.IN)
        # a short delay needed
        for i in range(10):
            pass

        # pullup by host 20-40 us
        count = 0
        while GPIO.input(self.pin):
            count += 1
            if count > self.MAX_CNT:
                # print("pullup by host 20-40us failed")
                set_default_priority()
                return None, "pullup by host 20-40us failed"

        pulse_cnt = [0] * (2 * PULSES_CNT)
        fix_crc = False
        for i in range(0, PULSES_CNT * 2, 2):
            while not GPIO.input(self.pin):
                pulse_cnt[i] += 1
                if pulse_cnt[i] > self.MAX_CNT:
                    # print("pulldown by DHT timeout %d" % i)
                    set_default_priority()
                    return None, "pulldown by DHT timeout %d" % i

            while GPIO.input(self.pin):
                pulse_cnt[i + 1] += 1
                if pulse_cnt[i + 1] > self.MAX_CNT:
                    # print("pullup by DHT timeout %d" % (i + 1))
                    if i == (PULSES_CNT - 1) * 2:
                        # fix_crc = True
                        # break
                        pass
                    set_default_priority()
                    return None, "pullup by DHT timeout %d" % i

        # back to normal priority
        set_default_priority()

        total_cnt = 0
        for i in range(2, 2 * PULSES_CNT, 2):
            total_cnt += pulse_cnt[i]

        # Low level ( 50 us) average counter
        average_cnt = total_cnt / (PULSES_CNT - 1)
        # print("low level average loop = %d" % average_cnt)

        data = ''
        for i in range(3, 2 * PULSES_CNT, 2):
            if pulse_cnt[i] > average_cnt:
                data += '1'
            else:
                data += '0'

        data0 = int(data[ 0: 8], 2)
        data1 = int(data[ 8:16], 2)
        data2 = int(data[16:24], 2)
        data3 = int(data[24:32], 2)
        data4 = int(data[32:40], 2)

        if fix_crc and data4 != ((data0 + data1 + data2 + data3) & 0xFF):
            data4 = data4 ^ 0x01
            data = data[0: PULSES_CNT - 2] + ('1' if data4 & 0x01 else '0')

        if data4 == ((data0 + data1 + data2 + data3) & 0xFF):
            if self._dht_type == self.DHT_TYPE['DHT11']:
                humi = int(data0)
                temp = int(data2)
            elif self._dht_type == self.DHT_TYPE['DHT22']:
                humi = float(int(data[ 0:16], 2)*0.1)
                temp = float(int(data[17:32], 2)*0.2*(0.5-int(data[16], 2)))
        else:
            # print("checksum error!")
            return None, "checksum error!"

        return humi, temp

    def read(self, retries = 15):
        for i in range(retries):
            humi, temp = self._read()
            if not humi is None:
                break
        if humi is None:
            return self._last_humi, self._last_temp
        self._last_humi,self._last_temp = humi, temp
        return humi, temp
```
</details>

<br />

### TDS水質センサ用クラス作成

`/home/pi/sensor/`に`grove_tds.py`を作成。

<details>
<summary>TDS水質センサ用クラス</summary>

```python:grove_tds.py
import math
import sys
import time
from grove.adc import ADC

class GroveTDS:

    def __init__(self, channel):
        self.channel = channel
        self.adc = ADC()

    @property
    def TDS(self):
        value = self.adc.read(self.channel)
        if value != 0:
            voltage = value*5/1024.0
            tdsValue = (133.42/voltage*voltage*voltage-255.86*voltage*voltage+857.39*voltage)*0.5
            return tdsValue
        else:
            return 0
```
</details>
<br />

### 実行プログラム作成

`/home/pi/`に`gardening_system.py`を作成。

```python:gardening_system.py
#!/usr/bin/env python3
import ambient
import sys
import time
import datetime
import schedule

from sensor.grove_tds import GroveTDS
from sensor.grove_temperature_humidity_sensor import DHT

DHT_TYPE = "11"
DHT_PIN = 12
TDS_CHANNEL = 0
AMBIENT_CHANNEL_ID = "-----"
AMBIENT_WRITE_KEY = "----------------"

def get_sensor_info():
    humi, temp = dht_sensor.read()

    data = {
        "created": datetime.datetime.now().strftime("%Y-%m-%d%H:%M:%S"),
        "d1": temp,
        "d2": humi,
        "d3": round(tds_sensor.TDS, 2)
    }
    try:
        am.send(data, timeout=10.0)
    except requests.exceptions.RequestException:
        pass

def main():
    schedule.every(1).minutes.do(get_sensor_info)

    while True:
        schedule.run_pending()
        time.sleep(1)

if __name__ == '__main__':
    dht_sensor = DHT(DHT_TYPE, DHT_PIN)
    tds_sensor = GroveTDS(TDS_CHANNEL)
    am = ambient.Ambient(AMBIENT_CHANNEL_ID, AMBIENT_WRITE_KEY)
    main()
```

以下の変数を作成したAmbientのチャネルのID、キーに置き換える。

`AMBIENT_CHANNEL_ID`、`AMBIENT_WRITE_KEY`

<br />

### プログラム自動起動設定

```bash
pi@raspberrypi:~ $ vi gardening-system.service
```

以下を記載。
```
[Unit]
Description=gardening-system daemon
[Service]
ExecStart=/usr/bin/python3 /home/pi/gardening_system.py
Restart=no
Type=simple
[Install]
WantedBy=multi-user.target
```

```bash
pi@raspberrypi:~ $ sudo mv gardening-system.service /etc/systemd/system/
pi@raspberrypi:~ $ sudo systemctl enable gardening-system.service
pi@raspberrypi:~ $ systemctl list-unit-files | grep gardening-system.service
pi@raspberrypi:~ $ sudo systemctl start gardening-system.service
pi@raspberrypi:~ $ systemctl status gardening-system.service
```

<br />

### Ambient動作確認

```bash
pi@raspberrypi:~ $ sudo reboot
pi@raspberrypi:~ $ systemctl status gardening-system.service
pi@raspberrypi:~ $ ps -ef | grep -v grep | grep gardening_system.py
```

Ambientで作成したチャネルでデータをグラフ化できていればOK。

- 温湿度センサ：0~50℃（±2℃）、20%~90%RH（±5%RH）

- TDS水質センサ：TDS(Total Dissolved Solids:総溶解固形物)

| TDSメーター | 水道水 | 浄水 |
| ------------- | ------------- | ------------- |
| TDS水質センサ  | 約215ppm | 約2~38ppm |
| TDS1(ハンナ製)  | 約124ppm | 約17ppm |

<br />

### 水耕栽培キット組み立て

説明書の通りに組み立てます。

<br />

### Raspberry Pi/水耕栽培キットの連結

TDS水質センサを水耕栽培キットの空いている穴に差してAmbientでデータが取れていることを確認。

<br />

### 参考にさせていただいたサイト

下記の記事を~~パクリ~~参考にさせていただきました。ありがとうございました！

[水耕栽培キットをRaspberry PiでIoT化してみる](https://zenn.dev/techkind/articles/2105291146_gardening_system)

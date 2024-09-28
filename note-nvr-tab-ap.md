## 通过平板电脑的AP通过wifi将平板电脑连接到录音机

该方案很简单：[TL-725n](https://www.tp-link.com/ru/home-networking/adapter/tl-wn725n/)或类似的rtp8188eu适配器插入录音机，或者OpenIPC 固件中有驱动程序的适配器；该平板电脑包括一个热点；记录仪连接到该接入点；重新启动后，wfb 服务会检测指定的 wlan 并配置对平板电脑的广播和遥测。

### Поднимем сеть
* Закачаем драйвер [8188eu](hi3536dv100/lib/modules/4.9.37/extra/8188eu.ko) в `/lib/modules/4.9.37/extra/`
* Настроим поднятие сети на адаптере в [`/etc/network/interfaces`](hi3536dv100/etc/network/interfaces), указывая свои ssid и password:
```
      auto wlan1
      iface wlan1 inet dhcp
          pre-up if ! lsmod | grep 8188eu; then insmod /lib/modules/4.9.37/extra/8188eu.ko; fi
          pre-up sleep 1
          pre-up wpa_passphrase "ssid" "password" >/tmp/wpa_supplicant.conf
          pre-up sed -i '2i \\tscan_ssid=1' /tmp/wpa_supplicant.conf
          pre-up sleep 3
          pre-up wpa_supplicant -B -D nl80211 -i wlan1 -c/tmp/wpa_supplicant.conf
          post-down killall wpa_supplicant
```
### Поправим конфиги сервисы
* Закачаем обновленные [`/usr/bin/wifibroadcast`](hi3536dv100/usr/bin/wifibroadcast) и [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry) с детектированием подключения в /usr/bin.
* Добавим в [wfb.conf](hi3536dv100/etc/wfb.conf) новую строчку с параметром - наименованием интерфейса для ap
```
tab_wlan=wlan1
```
### 让我们建立网络
* 下载驱动程序 [8188eu](hi3536dv100/lib/modules/4.9.37/extra/8188eu.ko) 到 `/lib/modules/4.9.37/extra/`
* 在[`/etc/network/interfaces`](hi3536dv100/etc/network/interfaces)中配置适配器上的网络提升，指定您的ssid和密码：
### 让我们修复服务配置
* 下载更新的 [`/usr/bin/wifibroadcast`](hi3536dv100/usr/bin/wifibroadcast) 和 [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry)，并在 /usr/bin 中进行连接检测。
* 在 [wfb.conf](hi3536dv100/etc/wfb.conf) 中添加一个带有参数的新行 - ap 的接口名称
* 如果我们不使用发送流到PC，我们可以注释掉`udp_addr`参数，这会减轻记录器的负担。
* 打开平板电脑上的接入点并重新启动录音机，或按[前面板](nvr_gpio.md)上的按钮。


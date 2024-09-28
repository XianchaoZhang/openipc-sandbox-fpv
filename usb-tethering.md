## 与 Android 上的 FPV-VR 和 GS 与 NVR 与 USB 兼容

USB 网络共享，即 USB 以太网 - ОС Android 版本与 USB-модема、представляющегос сетевой картой 相同。 Нам это дает возможность соединить телефон 和 NVR обычным USB-проводом 和 получить между ними быструю сеть。 К сожалению, в большинстве случаев USB terhering доступен только на планшетах с модулем sim-карты。 Если у вас его нет、usb-модем не активируется、придется подключаться по [usb-ethernet](usb-eth-modem.md) 和 [wifi](note-nvr-tab-ap.md)。

Нужно [установить](notes_start_hi3536ev100.md#L47) [ядро](hi3536dv100/uImage.hi3536dv100) и [rootfs](hi3536dv100/rootfs.squashfs.hi3536dv100) с поддержкой rndis_host и залить [`/usr/bin/wifibroadcast`](hi3536dv100/usr/bin/wifibroadcast), [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry), и добавить в [`interfaces`](hi3536dv100/etc/network/interfaces) следующее:
```
#USB Tethering
auto usb0
iface usb0 inet dhcp
    pre-up modprobe rndis_host
    pre-up sleep 4
```

Нужно [установить](notes_start_hi3536ev100.md#L47) [ядро](hi3536dv100/uImage.hi3536dv100) 和 [rootfs](hi3536dv100/rootfs.squashfs.hi3536dv100) с поде ржкой rndis_host 和 залить [`/usr/bin/wifibroadcast`]( hi3536dv100/usr/bin/wifibroadcast), [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry), и добавить в [`interfaces`](hi3536dv100/etc/network/interfaces) следующее: Для включения нужно: * Подключить смартфон или планшет кабелем к отключенном у регистратору * Зайти в настройки - Сеть и Интернет - Точка доступа и модем * Включить регистратор * Как только в меню Android 版本支持 USB 接口。
* Запустить FPV-VR, выбрать в настройках секцию Manually (stream), установить настройки видео на используемый кодек (rtp/264, rtp/265) и这是 mavlink 上的内容。
* Выйти из настроек и запустить“启动视频和 OSD”。

是的，我是 QOpenHD 的作者。

Также доступен [вариант без перезапуска](nvr_gpio.md) регистратора。


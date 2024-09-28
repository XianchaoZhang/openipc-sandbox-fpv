## 与 USB 以太网连接。

Устройство являет собой сетевую карту, подключаемую по USB к планшету, и выглядит примерно так:

![usb-ethernet.png](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/usb-ethernet.png)

Оно подключается к регистратору посредством ethernet-кабеля, после чего в [`/etc/wfb.conf`](hi3536dv100/etc/wfb.conf) указывается `udp_addr=auto` в замен фиксированного адреса。 На самом планшете нужно включить функцию “Ethernet-modem” в настройках “Сеть и интернет > Точка доступа и модем”。

Далее все сделают сервисы [`/usr/bin/wifibroadcast`](hi3536dv100/usr/bin/wifibroadcast) 和 [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry) при загрузке регистратора или перезапуске с [前面板](nvr_gpio.md)。


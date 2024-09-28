# sandbox-fpv 用于 FPV 实验的沙盒。电报群：`https://t.me/+BMyMoolVOpkzNWUy` | [链接](https://t.me/+BMyMoolVOpkzNWUy)

## 新闻
* `2023.07.26` - 通过 4G 调制解调器设置 FPV 链接。

* `01.07.2023` - 关于 imx335 gk7205v300 相机的简短说明。关于遥测的波特率。

* `2023.06.22` - 终于解决了 30fps 下画面卡顿的问题。

* `06.04.2023` - 为带有板载 u-boot 的 ivg-g2s 添加了耦合器固件。

* `05.04.2023` - 为 rcjoystick 添加了在 rssi 中显示数据包丢失（链路质量）的功能。

* `04.04.2023` - OpenIPC 添加了 majestic 流媒体，现在 ivg-g2s 相机运行 h265 cbr（恒定比特率）。这提供了更清晰的图像并显著降低了噪音。同时，对链接启动过程进行了更改。主要服务现在是 S98datalink，配置为 /etc/datalink.conf，wfb 现在通过 /usr/bin/wifibroadcast 启动。文章已更正以适应这一创新。

* `01.04.2023` - 由于某些情况，我的相机和录像机中的 wfb-ng 被 OpenHD 的替代品所取代。这是 buildroot OpenIPC 中用于组装的软件包。由于存在一个在 OpenHD 实现中不需要的参数 link_id，因此 shell 包装器考虑了这两个选项。包含两个选项的二进制文件的存档。

## Notes

* [关于在 gk7205v200 摄像头和 hi3536ev100 (dv100) 录像机上设置链接的说明](notes_link_gk7205v200_hi3536ev100.md)
* [关于 OpenIPC 上的摄像头固件 gk7205v200 的说明](notes_start_ivg-g2s.md)
* [关于 OpenIPC 上的 hi3536ev100 录像机固件的说明](notes_start_hi3536ev100.md)
* [关于 imx335 gk7205v300 摄像头的说明](notes_imx335_gk7205v300.md)
* [在 goke/hisilicon 摄像头上增加视频流畅度](gkrcparams.md)
* [关于通过地面遥控通道控制相机的注意事项](notes_cam_control.md)
* [在空中切换两个摄像头](note-two-cameras-switched.md)
* [gk7502v200 遥测加载程序，重启时不会挂起摄像头](gk7205v200_u-boot-7502v200-for-telemetry.md)
* [从录像机前面板控制按钮](nvr_gpio.md)
* [通过 USB 连接并设置平板电脑或智能手机以用于视频和 OSD](usb-tethering.md)
* [通过平板电脑的 AP 通过 wifi 将平板电脑连接到录像机](note-nvr-tab-ap.md)
* [通过 ethernet-usb-device 将平板电脑连接到录像机](usb-eth-modem.md)
* [使用硬件作为操纵杆进行传输通过 mavlink 连接 RC 频道](rcjoystick.md)
* [关于 RSSI 的模拟](rcjoystick.md#rssi)
* [SBUS 转 USB 操纵杆，用于使用任何带有 sbus 接收器的设备](sbus-to-usb-joystick)
* [通过 4G 调制解调器设置 FPV 链接](lte-fpv.md)
* [在带有 fpv、lite 固件的相机上安装 usb_modeswitch](usb-modeswitch.md)

#### 杂项
* [用于摄像头的 inav（单向 msp）的 mavfwd](user_TipoMan/mavfwd_mavlink2.tar?raw=true)
* [在 Windows 和 MP 上显示视频](gstlaunch_on_windows.md)
* [禁用 hi3536dv100 录像机上的看门狗](note_nvr_wdt.md)
* [与用于遥测的摄像头 uart 上的 115200 波特不同](note_telemetry_baud.md)

## 路线图
* ~~开始将视频从录像机传输到 PC。~~
* ~~启动单向和双向遥测。~~
* ~~开始通过 USB 网络共享将视频传输到 Android 平板电脑。~~
* ~~在 e3372h + zerotier 上构建和测试 LTE 固件~~
* ~~通过 mavlink-router 启动遥测路由。~~
* ~~寻找通过 mavlink 控制摄像头的方法。~~。
* 寻找通过 hdmi 输出视频和 osd 的方法。
* ~~~在多个摄像头之间切换，其中一个是带有 wfb-ng 的主摄像头，其余的都是从摄像头。~~~
* 开发摄像头扩展板：bec 5v/3.3v；usb 集线器、uart、wifi/调制解调器功率晶体管、microSD。
* 开发变焦镜头控制板和控制市售电路板的方法。
* 开发通过 wfb-ng 从地面控制的稳定万向架。



## Russians:

## 新闻
* `26.07.2023` - Настройка FPV-линка [через 4G модем](lte-fpv.md)。

* `01.07.2023` - Короткая заметка о камере [imx335 gk7205v300](notes_imx335_gk7205v300.md)。 О [波特率 телеметрии](note_telemetry_baud.md)。

* `22.06.2023` - Наконец [решилась](gkrcparams.md) проблема с дерганностью картинки на 30fps。

* `06.04.2023` - Добавлена [прошивка Coupler](notes_start_ivg-g2s.md#L33) для ivg-g2s с u-boot на борту.

* `05.04.2023` - В rcjoystick [добавлен](rcjoystick.md#rssi) функционал для целей отображения потерь пакетов (качествалинка) в rssi.

* `04.04.2023` - OpenIPC“допилили”стример Majestic，теперь на камере ivg-g2s работает h265 cbr (постоянный битрейт)。 Это дало более чистую картинку 和 значительное уменьшение шума. Вместе с этим были внесены изменения в процесс запуска линка。 Основным сервисом теперь является `S98datalink` с конфигом `/etc/datalink.conf`，以及 запуск wfb теперь производитсячерез `/usr/bin/wifibroadcast`。 Статьи были исправлены под это нововведение。

* `01.04.2023` - В связи с некоторыми обстоятельствами, wfb-ng был заменен в моих камере и регистраторе на альтернативу от [OpenHD]( https://github.com/OpenHD/wifibroadcast/）。 [Тут](wfbopenhd.zip) 在 buildroot OpenIPC 中运行。此链接包含 OpenHD 中的“link_id”和“link_id”。 [Архив](https://github.com/OpenIPC/sandbox-fpv/blob/master/wfb.zip) с бинарниками обоих вариантов。

## Заметки

* [Заметки о настройке линка на камере gk7205v200 和 регистраторе hi3536ev100 (dv100)](notes_link_gk7205v200_hi3536ev100.md) * [Заметки о прошивке камеры gk7205v200 на OpenIPC](notes_start_ivg-g2s.md) * [Заметки о прошивке регистратора hi3536ev100 на OpenIPC](notes_start_hi3536ev100.md) * [Заметка о камере imx335 gk7205v300](notes_imx335_gk7205v300.md) * [Добавляем плавности видео на goke/hisilicon ка мерах](gkrcparams.md) * [Заметка о управлении камерой через RC каналы с наземки](notes_cam_control .md) * [Переключение между двумя камерами в воздухе](note-two-cameras-switched.md) * [Загрузчик под телеметрию для gk7502v200, который не вешает камеру при ребуте](gk7205v200_u-boot- 7502v200-for-telemetry.md) * [Управление кнопками с 前面板 на регистраторе](nvr_gpio.md) * [Подключение и настройка планшета или смартфона для видео 和 OSD по USB](usb-tethering.md) * [Подключение планшета к регистратору по wifi через AP планшета](note-nvr-tab-ap.md) * [Покд] лючение планшета к регистратору через 以太网 USB 设备](usb-eth-modem.md) * [Использование аппаратуры как джойстика для передачи каналов RC через mavlink](rcjoystick.md) * [Про аналог RSSI](rcjoystick.md#rssi) * [SBUS 转 USB 操纵杆 для использования любой аппаратуры с sbus приемником](sbus-至 USB 操纵杆）* [ Настройка FPV-линка через 4G модем](lte-fpv.md) * [Установка usb_modeswitch на камеру с прошивкой fpv, lite](usb-modeswitch.md)

#### Разное * [mavfwd для inav (односторонний msp) для камеры](user_TipoMan/mavfwd_mavlink2.tar?raw=true) * [Отображение видео на windows и в MP](gstlaunch_on_windows.md) * [ hi3536dv100 上的看门狗](note_nvr_wdt.md) * [Отличный от 115200 波特 на uartе камеры для телеметрии](note_telemetry_baud.md)

## Дорожная карта * ~~Запуск видео с передачей с регистратора на пк.~~ * ~~Запуск одно-и двусторонней телеметрии.~~ * ~~Запуск已在 android-планшет 上使用 USB 网络共享。~~ * ~~Сборка和 e3372h + Zerotier 上的 LTE ~~ * ~~Запуск маршрутизации телеметрии через mavlink-router.~~ * ~~Поиск путей управления камерой сквозь mavlink ~~.
* 播放视频和 osd 屏幕 HDMI。
* ~~~Переключение между несколькими камерами, где одна ведущая с wfb-ng, а остальные ведомые.~~~ * Разработка платы расширения для камеры: bec 5v/3.3v; USB 集线器、UART、无线网络/调制解调器、microSD。
* Разработка платы управления зум-объективом 和 способа управления имеющимися в продаже платами.
* Разработка стабилизирующего подвеса, управляемого с земли сквозь wfb-ng。


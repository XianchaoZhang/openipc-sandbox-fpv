# sandbox-fpv
Sandbox for FPV experiments. Telegram-group: [link](https://t.me/+BMyMoolVOpkzNWUy)

## 新闻
* `2023.07.26` - 通过 4G 调制解调器设置 FPV 链接。

* `01.07.2023` - 关于 imx335 gk7205v300 相机的简短说明。关于遥测的波特率。

* `2023.06.22` - 终于解决了 30fps 时画面卡顿的问题。

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
* ~~在多个摄像头之间切换，其中一个是带有 wfb-ng 的主摄像头，其余的都是从摄像头。~~
* 开发摄像头扩展板：bec 5v/3.3v；usb 集线器、uart、wifi/调制解调器功率晶体管、microSD。
* 开发变焦镜头控制板和控制市售电路板的方法。
* 开发通过 wfb-ng 从地面控制的稳定万向架。



## Russians:

## 新闻
* `07.26.2023` - 设置 FPV 链接 [通过 4G 调制解调器](lte-fpv.md)。

* `07/01/2023` - 关于相机的简短说明 [imx335 gk7205v300](notes_imx335_gk7205v300.md)。关于[遥测波特率](note_telemetry_baud.md)。

* `06/22/2023` - 终于[解决了](gkrcparams.md) 30fps 时图片抖动的问题。

* `04/06/2023` - 为带有 u-boot 的 ivg-g2s 添加了[情侣固件](notes_start_ivg-g2s.md#L33)。

* `04/05/2023` - 在 rcjoystick 中[添加](rcjoystick.md#rssi) 功能，用于在 rssi 中显示数据包丢失（链接质量）。

* `04/04/2023` - Majestic Streamer 已添加到 OpenIPC，现在 h265 cbr（恒定比特率）适用于 ivg-g2s 相机。这提供了更清晰的图像并显着减少了噪音。同时，链接启动流程也发生了变化。主要服务现在是带有配置"/etc/datalink.conf"的"S98datalink"，并且 wfb 现在通过"/usr/bin/wifibroadcast"启动。这些文章已被更正以适应这一创新。

* `04/01/2023` - 由于某些情况，我的相机和记录仪中的 wfb-ng 被 [OpenHD](https://github.com/OpenHD/wifibroadcast/) 的替代品替换。 [此处](wfbopenhd.zip) 用于在 buildroot OpenIPC 中进行组装的包。 shell 包装器通过存在"link_id"参数来考虑这两个选项，而 OpenHD 实现中不需要该参数。 [存档](https://github.com/OpenIPC/sandbox-fpv/blob/master/wfb.zip) 以及两个选项的二进制文件。

## Notes

* [关于在 gk7205v200 摄像机和 hi3536ev100 (dv100) 录像机上设置链接的注意事项](notes_link_gk7205v200_hi3536ev100.md)
* [OpenIPC 上相机 gk7205v200 固件说明](notes_start_ivg-g2s.md)
* [OpenIPC上hi3536ev100记录仪固件说明](notes_start_hi3536ev100.md)
* [关于相机imx335 gk7205v300的说明](notes_imx335_gk7205v300.md)
* [为国科/海思相机上的视频添加平滑度](gkrcparams.md)
* [关于从地面通过RC通道控制相机的注意事项](notes_cam_control.md)
* [空中切换两个摄像头](note-two-cameras-switched.md)
* [gk7502v200遥测引导加载程序，重启时不会挂起相机](gk7205v200_u-boot-7502v200-for-telemetry.md)
* [从记录仪前面板管理按钮](nvr_gpio.md)
* [通过 USB 连接和设置平板电脑或智能手机以进行视频和 OSD](usb-tethering.md)
* [通过平板电脑的AP通过wifi连接平板电脑和录像机](note-nvr-tab-ap.md)
* [通过 ethernet-usb-device 将平板电脑连接到记录仪](usb-eth-modem.md)
* [使用设备作为摇杆通过mavlink传输RC通道](rcjoystick.md)
* [关于 RSSI 模拟](rcjoystick.md#rssi)
* [SBUS 转 USB 操纵杆，用于使用带有 sbus 接收器的任何设备](sbus-to-usb-操纵杆)
* [通过4G调制解调器设置FPV链接](lte-fpv.md)
* [在带有 fpv、lite 固件的相机上安装 usb_modeswitch](usb-modeswitch.md)

#### 杂项
* [用于相机的 inav（单向 msp）的 mavfwd](user_TipoMan/mavfwd_mavlink2.tar?raw=true)
* [在 Windows 上和 MP 中显示视频](gstlaunch_on_windows.md)
* [禁用hi3536dv100录像机上的看门狗](note_nvr_wdt.md)
* [与遥测相机uart上的115200波特不同](note_telemetry_baud.md)

## 路线图
* ~~启动视频并从录像机传输到电脑。~~
* ~~启动单向和双向遥测。~~
* ~~开始通过 USB 网络共享将视频传输到 Android 平板电脑。~~
* ~~在 e3372h + Zerotier 上构建和测试 LTE 固件~~
* ~~通过mavlink-router启动遥测路由。~~
* ~~正在寻找通过mavlink控制相机的方法~~。
* 寻找通过hdmi输出视频和OSD的方法。
* ~~多台相机之间切换，其中一台用wfb-ng为主，其余为从。~~
* 开发摄像头扩展板：bec 5v/3.3v； USB 集线器、UART、wifi/调制解调器功率晶体管、microSD。
* 开发变焦镜头控制板和市售板的控制方法。
* 开发通过 wfb-ng 从地面控制的稳定万向节。


## 通过 USB 将用于 FPV-VR 和其他 GS 程序的 Android 智能手机或平板电脑连接到 NVR

USB 网络共享，也称为 USB 以太网，是 Android 操作系统的一项功能，可将手机用作 USB 调制解调器，伪装成网卡。这使我们有机会使用普通 USB 电缆连接手机和 NVR，并在它们之间建立快速网络。不幸的是，在大多数情况下，USB 网络共享仅适用于带有 SIM 卡模块的平板电脑。如果您没有它并且 USB 调制解调器未激活，则必须通过 [usb-ethernet](usb-eth-modem.md) 或 [wifi](note-nvr-tab-ap.md) 连接。

Нужно [установить](notes_start_hi3536ev100.md#L47) [ядро](hi3536dv100/uImage.hi3536dv100) и [rootfs](hi3536dv100/rootfs.squashfs.hi3536dv100) с поддержкой rndis_host и залить [`/usr/bin/wifibroadcast`](hi3536dv100/usr/bin/wifibroadcast), [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry), и добавить в [`interfaces`](hi3536dv100/etc/network/interfaces) следующее:
```
#USB Tethering
auto usb0
iface usb0 inet dhcp
    pre-up modprobe rndis_host
    pre-up sleep 4
```

您需要 [install](notes_start_hi3536ev100.md#L47) [kernel](hi3536dv100/uImage.hi3536dv100) 和 [rootfs](hi3536dv100/rootfs.squashfs.hi3536dv100) 以及 rndis_host 支持并填写 [`/usr/bin/wifibroadcast ` ]( hi3536dv100/usr/bin/wifibroadcast), [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry), 并将以下内容添加到 [`interfaces`](hi3536dv100/etc/network/interfaces )：要启用需要：
* 使用电缆将您的智能手机或平板电脑连接到断开的录音机
* 转到设置 - 网络和 Internet - 接入点和调制解调器
* 启用录音机
* 一旦 USB 调制解调器项目在 Android 菜单中显示为活动状态，请将其打开。
* 启动 FPV-VR，在设置中选择手动（流）部分，将视频设置设置为使用的编解码器（rtp/264、rtp/265），将遥测设置设置为 mavlink。
* 退出设置并运行“启动视频和 osd”。

或者，您可以使用具有类似功能的 QOpenHD 应用程序。

还可以使用记录器的[无需重新启动的选项](nvr_gpio.md)。


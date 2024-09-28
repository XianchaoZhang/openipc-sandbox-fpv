## 通过 USB 以太网设备连接平板电脑或手机。

该设备是一个通过 USB 连接到平板电脑的网卡，看起来像这样：

![usb-ethernet.png](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/usb-ethernet.png)

通过以太网线连接到录像机，然后在[`/etc/wfb.conf`](hi3536dv100/etc/wfb.conf)中指定`udp_addr=auto`而不是固定地址。在平板电脑本身上，您需要在"网络和互联网 > 接入点和调制解调器"设置中启用"以太网调制解调器"功能。

接下来，加载时一切都将由服务 [`/usr/bin/wifibroadcast`](hi3536dv100/usr/bin/wifibroadcast) 和 [`/usr/bin/telemetry`](hi3536dv100/usr/bin/telemetry) 完成记录仪或从[前面板](nvr_gpio.md)重新启动。


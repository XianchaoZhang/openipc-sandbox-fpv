## 使用硬件作为摇杆通过 mavlink 传输 RC 通道

### 理论
地面站程序，例如 Mission Planner或QGroundControl，能够识别连接的操纵杆或USB操纵杆模式的设备，并将轴值传输到RC_CHANNELS_OVERRIDE（#70）包中的飞控。然而，QGC 只传输前 4 个通道（两个轴），其余的只能分配给按钮，而 MP 只能在 Windows 下执行此操作，这并不总是很方便。我为记录仪编写了一个简单的应用程序，它可以识别连接的操纵杆并以 mavlink 协议版本 2 发送 [RC_CHANNELS_OVERRIDE](https://mavlink.io/en/messages/common.html#RC_CHANNELS_OVERRIDE) 数据包，以支持直接将 18 个通道发送到telemetry_tx 端口，通常发布在注册商 127.0.0.1:14650 上。系统中的端口、地址、发送数据包的时间间隔、轴数和设备都可以重新分配，请参见"rcjoystick -h"。

```
Usage:
 [-v] verbose;
 [-d device] default '/dev/input/js0';
 [-a addr] ip address send to, default 127.0.0.1;
 [-p port] udp port send to, default 14650;
 [-t time] update RC_CHANNEL_OVERRIDE time in ms, default 50;
 [-x axes_count] 2..9 axes, default 5, other channels mapping to js buttons from button 0;
 [-r rssi_channel] store rx packets per second value to this channel, default 0 (disabled);
 [-i interface] wlan interface for rx packets statistics, default wlan0;

```

例如，如果需要，您可以不发送到 telemetry_tx，而是发送到 mavlink_routerd。用于从 OpenIPC [此处](rcjoystick) 在 buildroot 中构建的包。

### 开始 
我们需要一个内核和 rootfs，并在刻录机上支持 USB-hid。为此，请从 [`/hi3536dv100`](hi3536dv100) 目录中 [flash](notes_start_hi3536ev100) 它们。 ~~需要确保启动 `hid-generic.ko` 模块才能执行此操作，将 `modprobe hid-generic.ko` 添加到 [`S95hisilicon`](hi3536dv100/etc/init.d/S95hisilicon) .~~ 在新编译的内核中，会自动加载 ` module hid-generic.ko` 。

接下来，您需要通过 WinSCP 将 [`rcjoystick`](hi3536dv100/usr/bin/rcjoystick) 二进制文件复制到记录仪的 /usr/bin 并分配执行权限：`chmod +x /usr/bin/rcjoystick`。我们重新启动，通过 USB 将设备连接到记录仪，并尝试在控制台中运行"rcjoystick -v"。如果一切顺利，那么在更改摇杆和开关位置时，我们应该在屏幕上看到轴值，并且在遥测程序中，例如在 QGC 中（分析工具 > Mavlink 检查器 > RC_CHANNELS_RAW），通道应该更改。要永久启动它，您可以在单独的服务[`S99rcjoystick`](hi3536dv100/etc/init.d/S99rcjoystick)中注册它，主要是它在wifibroadcast之后启动。

### RSSI

此外，还向 rcjoystick 添加了一个功能，用于注入 rssi 类似物（每秒从机载部分接收到的数据包数量），至少用于接收质量的某些指标，因为没有必要或没有必要为以下内容创建单独的服务：鉴于记录仪的电平较低。 rcjoystick 已经具备了注射所需的大部分内容。

我们指定"-r 16"，如果wfb接口不是wlan0，那么它的"-i wlanX"和land每秒接收的数据包数量将落入指定的通道。我的数量大约为 800，您可以使用详细开关 (`-v`) 查看您的数量。接下来，我们在飞行的飞机上注明：

RSSI_TYPE	2
RSSI_CHANNEL	16
RSSI_CHAN_LOW	0
RSSI_CHAN_HIGH	800 // 您的正常值大约是平均值，既不是最大值也不是最小值。
```
该值将被发送到飞行管理器，在那里它将对其进行处理并在 rssi 中设置 0..99。

该项目仍处于测试阶段，因此[发送](https://t.me/+BMyMoolVOpkzNWUy)您对尝试和建议的反馈。

### 当前问题 
如果您向各个方向积极按下两根摇杆，有时会观察到短期冻结。德巴格表示，目前驾驶员没有发出任何动作事件。如果您使用 arduino pro micro 形式的[操纵杆的"无线电扩展器"](sbus-to-usb-操纵杆)，则不会观察到这种效果，它会解析来自接收器的 SBUS 并将其转换为 USB-隐藏操纵杆。


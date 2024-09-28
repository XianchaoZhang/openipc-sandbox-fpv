
### 使用双向遥测 mavlink 构建视频链路的注意事项

链路的空中部分是一个相机 [gk7205v200](https://sl.aliexpress.ru/p?key=e1sTwWg)，带有通过 rtl8812au 芯片连接的 USB wifi 适配器，例如华硕 USB AC-56 或 [ ali 的较弱适配器](https://sl.aliexpress.ru/p?key=8CsTwDB)。地面部分是基于海思 hi3536dv100 或 ev100 芯片的 [DVR](https://sl.aliexpress.ru/p?key=L1sTwWG)，还通过 USB 连接 rtl8812au 或 rtl8814au 适配器。对于俄罗斯联邦，从 [@ser177](https://t.me/ser177) 订购摄像机和录像机更便宜、更快捷。本文介绍了创建此类链接的细微差别，并且是对[本文](https://github.com/OpenIPC/wiki/blob/master/ru/fpv.md)的补充。

![link_hw](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/link_hw.png)

### 可能性

此链路能够使用 h264 或 h265 编解码器和 mavlink 从机载部分传输视频 [(youtube)](https://youtu.be/ldfQ9CLE86I)，格式为（分辨率@帧速率）1920x1080@30 或 1280x720@50两个方向的遥测。视频传输的一般流程图如下：

![video](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/video.png)

目前可以通过两种方式进行视频传输。经典 - 流媒体 `majestic` 通过 udp 端口​​ 5600 将 RTP/h264 或 RTP/h265 流发送到 [wfb-ng](https://github.com/svpcom/wfb-ng) 或 [OpenHD-wfb](https:// /github.com/OpenHD/wifibroadcast）用于传输到地面，由 wfb 响应接收并通过 LAN 或 USB 以太网（网络共享）发送到 PC 或平板电脑/智能手机进行播放。 RTP 格式可以由GS程序自由复制，例如 [QGroundControl](https://github.com/mavlink/qgroundcontrol)、[Mission Planner](https://ardupilot.org/planner/)、[QOpenHD](https : //openhdfpv.org/download/) 或 [FPV-VR](https://github.com/Consti10/FPV_VR_OS)。但目前还无法将其输出到录像机的 HDMI 端口，因为它是在带有自己的 SDK 的专用芯片上构建的，并且这不能以通常的方式完成，例如通过 GStreamer，像往常一样，视频是Raspberry Pi 的情况下的输出。

<sub>QGroundControl 在播放 h265 时有一个 bug，表现为进入菜单时图像冻结。通过选择视频流 h264 并返回 h265，可以在重新启动程序之前解决此问题。</sub>

OpenIPC 团队的 Andrey Bezborodov 提供了 *vencoder* 和 *vdecoder* 的编译示例用于测试，这些示例是从 Hisilicon SDK 中提取的，并以其原始形式位于[此处](https://github.com/OpenIPC/silicon_research)。 `venc` 在摄像机上运行，​​生成带有 HAL 碎片的 h264 流，而不是 `majestic`，录像机上的 `vdec` 将此流输出到 HDMI。一切正常，但当然没有 OSD，第三方播放器无法再现这种非标准流。这是一种非常有前途的方式，因为它有可能减少视频传输延迟。目前它的范围是 110 到 130 毫秒。在"经典方案"中，延迟通常为 150 到 230 毫秒，[这里是 133 毫秒的示例](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/Screenshot_1.png)，具体取决于分辨率和地面播放条件。

这个问题可以通过使用 *libavformat/avformat.h* 库"教" `vdec` 识别 RTP/h26[4-5] 并仍然通过 `majestic`在相机上"流"来解决。如果您想帮助该项目，这需要 C++ 程序员的帮助 - [联系](https://t.me/+BMyMoolVOpkzNWUy)。

在 Mission Planner 上设置 *GStreamer* 以播放 h265 的示例： `udpsrc port=5600 buffer-size=32768 ! application/x-rtp ! rtph265depay ! avdec_h265 ! videoconvert ! video/x-raw,format=BGRA ! appsink name=outsink`

atheros 2.3 - 2.4ghz 上支持 5.2ghz 到 5.85ghz 的频率。

### 一切是如何开始的？

当linux启动时，`S98datalink` 服务会从 `init.d` 的服务中启动，这是起点。它运行脚本 `/usr/bin/wifibroadcast`，该脚本通过 lsusb 确定连接了哪个适配器，加载其驱动程序，切换到监视模式，启动 `wfb` 进行传输或接收，对于地面，它确定通过 USB 的连接，第二个 wifi 适配器或刚刚开始将视频传输到 `udp_addr`。它从 `/etc/wfb.conf` 获取设置数据。此外，当启用遥测时，它会运行脚本 `/usr/bin/telemetry`，该脚本执行相同的操作，但出于遥测目的，从 `/etc/telemetry.conf` 获取设置。

### 相机上的细微差别

该相机有两个传感器驱动程序 - "慢速"1080p@30fps 和"快速"720p@50fps。如果您上传 ["快速"驱动程序]，可以使用 [root](https://github.com/OpenIPC/sandbox-fpv/tree/master/gk7205v200/root) 中的示例脚本即时切换它们(gk7205v200/lib) 到相机 /sensors/libsns_imx307_2l_720p.so) 以单独的名称并在其配置中更正其路径 [`imx307_i2c_2l_720p_50fps.ini`](gk7205v200/etc/sensors/imx307_i2c_2l_720p_50fps.ini# L15）。该相机的所有文件都位于"gk7205v200"目录中。如果您在雄伟设置中使用"快"驱动程序启动相机，视频传输会很不稳定，因此当您通过"S95goke"启动相机时，"慢"驱​​动程序的设置会被注册，之后您就可以打开就"快"而言。目前[工作正在进行中](notes_cam_control.md) 通过 mavlink 中的 RC 通道管理此类相机设置。

### 录像机的细微差别
由于摄像机具有华丽的 16 MB spi 闪存，其中我们可以使用大约 5 MB，[RTL 适配器驱动程序](https://github.com/OpenIPC/sandbox-fpv /tree）可供我们使用/master/hi3536dv100/88XXau-ko），除了流行的 rtl8812au 之外，它还支持 rtl8814au。为此，您需要将其上传到 `lib/modules/4.9.37/extra` 中的标准版本之上，不要忘记重命名它。

重新编译 [`mavlink-router`](https://github.com/OpenIPC/sandbox-fpv/tree/master/hi3536dv100/usr/bin)，因为完整的固件是在 musl 中组装的空中部分(在哪里)未使用），记录仪固件基于glibc。

还需要[禁用海思看门狗](note_nvr_wdt.md)。

### 遥测的细微差别 
当前的遥测方案如下所示：

![telemetry](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/telemetry.png)

~~通过使用 mavlink-routerd，目前只能通过 udp 进行单向遥测，因为它不能按照 wfb 的要求在同一端点内使用不同的 rx/tx udp 端口​​，由不同的进程 `telemetry_rx` 启动，并且`telemetry_tx`。 ~~ 由不同的进程启动，`telemetry_rx` 和 `telemetry_tx` 使用不同的端口来接收和传输数据（顺便说一句，这些只是由 [telemetry launch] 脚本创建的 wfb_rx 和 wfb_tx 的符号链接（ hi3536dv100/usr/bin/telemetry)) ，并且 mavlink-router 在 [配置](hi3536dv100/etc/mavlink.conf) 中需要两个必须分组的 UDP 端点：
```
[UdpEndpoint telemetry_tx]
Group=wfb
Mode = Normal
Address = 127.0.0.1
Port = 14550

[UdpEndpoint telemetry_rx]
Group=wfb
Mode = Server
Address = 127.0.0.1
Port = 14551
```

其余端点需要与地面站通信，用于例如 tcp:5760 用于接收来自 Mission Planner 的连接。对于UDP链接设置中的[附加配置](hi3536dv100/etc/mavlink.conf)，您需要指定注册器地址：

![udp-qgc](notes_files/qgc-udp-settings.png)

剩下的就是切换到 /usr/bin/telemetry 以使用 mavlink-routerd，并且您不再需要连接注册器的 uart。

```
  /usr/bin/mavlink-routerd -c /etc/mavlink.conf &
  #/usr/sbin/mavfwd --master ${serial} --baudrate ${baud} --out 127.0.0.1:${port_tx} --in 127.0.0.1:${port_rx} &
```

如果要使用 uart，可以将端点设置为 /dev/ttyAMA0 或切换到 mavfwd。在这种情况下，您需要通过注释掉以下行来禁用 /etc/inittab 中 uart 的 ssh 控制台：

```
#console::respawn:/sbin/getty -L  console 0 vt100 # GENERIC_SERIAL
```
然后，遥测将在记录仪的 uart 上可用，而不是通过网络的 udp，或者除了网络上的 udp 之外，还可以通过 USB-uart 适配器将其用作串行端口。在QGC上，连接串口需要提前关闭流控，否则会加载一半左右的参数并报错。

我为支持 b230400、b500000、b921600 和 b1500000 速度的记录器编译了 "mavfwd"，以在运行 Mission Planner 时支持更高的速度，并在相机上使用 b115200/b230400 在 b500000 上进行了测试。对于记录仪，您可以在[此处](hi3536dv100/usr/sbin)获取。对于相机：[此处](https://github.com/OpenIPC/sandbox-fpv/tree/master/gk7205v200/usr/sbin)。 [来源](https://github.com/OpenIPC/sandbox-fpv/tree/master/mavfwd)。相机能够以230400的速度与飞行员建立稳定的连接，但无法高于stm32f4。设置 ardupilot 的飞行速度是使用"SERIALx_BAUD"参数完成的，在我的例子中："230"。另外，不要忘记将"TELEM_DELAY"参数设置为 10（遥测开始前的秒延迟），否则遥测可能会停止引导加载程序。不幸的是，如果在飞行过程中相机因某种原因与飞行员分开重新启动，那么遥测~~将不允许它加载~~可能不允许它加载。 ~~需要修改u-boot加载程序，使其不会在任何字符处停止加载。~~ ~~C [new u-boot](gk7205v200_u-boot-7502v200-for-telemetry.md),~~仅通过Ctrl+C中断，并且`bootdelay=0`测试中不存在此问题。此 U-boot 已包含在所有 OpenIPC FPV 固件中。

我还构建了一个[框架](notes_cam_control.md)用于监视选定的 RC mavlink 通道，并在更改为 `/root/channels.sh` 时将其值作为参数$1（通道）和$2（值）传递的基础。

### 当前问题
如果在将摄像机加载到 Majestic 中时选择"快速"720p 驱动程序，则视频会不稳定，因此在 S95goke（Majestic 自动启动）中，在启动之前安装"常规"1080p 驱动程序。如果您想在加载相机时默认使用 720p@50，请在 [`/etc/init.d/S95majestic`](gk7205v200/etc/init.d/S95majestic#L35) 中插入对切换脚本的调用`load_majestic 加载 Majestic 后的函数`:
```
yaml-cli -s .isp.sensorConfig /etc/sensors/imx307_i2c_2l_1080p.ini
yaml-cli -s .video0.size 1920x1080
yaml-cli -s .video0.fps 30

start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "/usr/bin/$DAEMON" \
	-- $DAEMON_ARGS
status=$?
if [ "$status" -eq 0 ]; then
	echo "OK"
else
	echo "FAIL"
fi

sleep .5
/root/720.sh

return "$status"
```


## 在记录仪上配置具有自己功能的按钮

![前面板](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/IMG_20230323_081622_212.jpg)

记录仪板上有一个连接器，用于连接带按钮的前面板和红外接收器：

![nvr 端口](notes_files/photo_2023-03-23_02-12-40.jpg)

cn5 连接器的标签位于背面。分配+3.3V和GND后，很明显无法使用IR，其余引脚通向处理器的GPIO：
```
Y2    ^17
Y1    ^6
X2    ^13
Y3    ^8
X1    ^7
ALARM 10
REC   11
```

符号"^"表示电阻上拉至+3.3，即意味着这些引脚必须用接地按钮关闭，并且值 0 被捕获在文件 [ `root/gpio_monitor.sh`](hi3536dv100/root/gpio_monitor.sh) 中。通过将引脚 Y1 接地，它会重新启动服务 [wfb](hi3536dv100/etc/init.d/S98wfb)，然后重新启动 [遥测](hi3536dv100/usr/bin/telemetry)，以便更方便地连接智能手机或平板电脑 [通过 USB](usb-tethering.md)，或更改 wifi 适配器后。监控脚本保存点击日志，可以使用"tail -f /tmp/gpio.log"进行观察。使用 GPIO 输出的示例可以在 [`testgpio.sh`](hi3536dv100/root/testgpio.sh) 中找到，您可以将 ALARM 或 REC 引脚连接到带有电阻的低功耗 LED 来指示进程，例如按照 `gpio_monitor.sh` 中的操作重新启动 wfb-ng。

要将监视器作为系统守护进程启动，我们将创建一个文件 [`/etc/init.d/S99gpio_monitor`](hi3536dv100/etc/init.d/S99gpio_monitor)，从中启动我们的 [`root/gpio_monitor.sh `](hi3536dv100/root/gpio_monitor.sh)：
```
#!/bin/sh
#
# Start gpio monitor
#

case "$1" in
  start)
    echo "Starting gpio_monitor daemon..."
    /root/gpio_monitor.sh &
    ;;
  stop)
    echo "Stopping gpio_monitor daemon..."
    kill -9 $(pidof {exe} ash /root/gpio_monitor.sh)
    ;;
    *)
    echo "Usage: $0 {start|stop}"
    exit 1
esac
```

在没有 wifi 适配器和/或 USB 调制解调器的情况下重新启动，加载后激活它们，并确保当您按下按钮（按住至少半秒）时，服务启动。您始终可以使用命令"ps axww"查看正在运行的进程列表。


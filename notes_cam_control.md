### 从地面控制相机

作为实验，[`mavfwd`](mavfwd) 补充了 mavlink 包 RC_CHANNELS 的解析器（从任何 RC 链路发送的 RC 通道值，例如从地面站 [(joystick)](https:// github.com/whoim2/ arduremote）通过 [Mission Planner 操纵杆](https://ardupilot.org/copter/docs/common-joystick.html)) 或从 [connected](rcjoystick.md) 到操纵杆记录器。

它监视 `--channels X` 或 `-c X` 参数中指定的通道的变化，在前 4 个之后计数，如果有，则调用脚本 [`/root/channels.sh`](gk7205v200 /root），传递给它有两个参数（通道号和值），并执行必要的操作。例如，“-c 1”将仅监视通道 5，而“-c 4”将监视通道 5、6、7、8。在当前的[示例](gk7205v200/root/channels.sh)中，这是更改第5个通道上的相机操作模式(1080p@30fsp / 720p@5-fps)，更改第7个通道上的亮度(三位开关)和 ircut 开关（偏振滤光片）。默认为“-c 0”，即禁用。

要将其安装到相机上，请将`/usr/sbin/`中的标准mavfwd替换为[modified](mavfwd/mavfwd)，并将`-c`参数添加到[`/usr/bin/telemetry`](gk7205v200 /usr/bin/遥测#L39）。同时，您将有机会将与飞行员通信的遥测速度设置为高于115200，当然后果自负！欢迎就此事提出想法和建议，并可以在[此处](https://t.me/+BMyMoolVOpkzNWUy)表达。

upd 03/31/2023 添加了对 mavlink 2 协议的识别。


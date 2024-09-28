## 为视频添加流畅度
在 1080p@30fps 模式下，视频有轻微的抽搐现象，当使用定时器进行慢动作拍摄时，您可以看到画面冻结了一段时间，然后更新。这是由于流量的不均匀性，在关键帧处急剧增加。这可以通过重新配置相机上的两个编码器参数来纠正。感谢 TipoMan 和 widgetii 所做的工作！

我们需要将文件 [gkrcparams](https://github.com/OpenIPC/sandbox-fpv/raw/master/user_TipoMan/gkrcparams) 放在 /usr/sbin 中，并赋予执行权限 `chmod +x /usr/sbin/ gkrcparams ` 并在 /etc/init.d/S95majestic 中启动 Majestic 之后插入其启动：

```
	start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "/usr/bin/$DAEMON" -- $DAEMON_ARGS
	sleep 1                        <=== ЭТО ВСТАВИТЬ
	gkrcparams --MaxQp 30 --MaxI 2 <=== ЭТО ВСТАВИТЬ
	status=$?
```
重新启动后，画面应该变得流畅。 majestic.yaml 中 mcs1 模式的其他设置：

```
video0:
  enabled: true
  bitrate: 7168
  codec: h265
  rcMode: cbr
  gopSize: 1.0
  size: 1920x1080
```

如果图像有时仍会抽搐，则必须在"/etc/wfb.conf"中将 mcs 更改为 3，从而失去范围或降低比特率。


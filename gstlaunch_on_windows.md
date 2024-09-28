## 在 Windows 上接收并显示视频

在"经典模式"下（摄像机上的 majestic 码流，NVR 将视频流发送到 PC 上，不要忘记在录像机的 wfb.conf 中注明 PC 的 IP 地址），可以接收视频在 QGroundControl、Mission Planner 中，可以简单地显示在单独的窗口中，而无需与程序绑定。为此，您需要安装 [GStreamer](https://gstreamer.freedesktop.org/download/) 并运行它以使用某些参数进行接收，例如： 

```
C:\gstreamer\1.0\msvc_x86_64\bin\gst-launch-1.0.exe -v udpsrc port=5600 buffer-size=32768 ! application/x-rtp ! rtph265depay ! queue max-size-buffers=5 ! avdec_h265 ! videoconvert ! videoscale ! video/x-raw,width=1280,height=720,format=BGRA ! autovideosink sync=false
```

在本例中，视频尺寸更改为 1280x720。要启动具有原始流分辨率的视频，请从该行中删除 `videoscale ! ` 和 `width=1280,height=720,`。


![preview](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/Screenshot_2.png)

要在 Mission Planner 窗口中播放视频，您需要右键单击其带有地平线的窗口，然后选择 `Video > Set GStreamer source`，输入参数行： `udpsrc port=5600 buffer-size=32768 ! application/x-rtp ! rtph265depay ! queue max-size-buffers=5 ! avdec_h265 ! videoconvert ! video/x-raw,format=BGRA ! appsink name=outsink`，单击确定。该字符串将被保存以供将来使用。


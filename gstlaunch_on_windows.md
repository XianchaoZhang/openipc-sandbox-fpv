## Windows 上的示例和视频

В "классическом режиме" (на камере стримит majestic, а NVR пересылает видеопоток на ПК, не забываем указать IP адрес ПК в wfb.conf регистратора) видео можно принимать в QGroundControl, Mission Planner и можно просто вывести в отдельном окне
без привязки к программам. Для этого нужно установить [GStreamer](https://gstreamer.freedesktop.org/download/) и запускать его на прием с некими параметрами, например:
```
C:\gstreamer\1.0\msvc_x86_64\bin\gst-launch-1.0.exe -v udpsrc port=5600 buffer-size=32768 ! application/x-rtp ! rtph265depay ! queue max-size-buffers=5 ! avdec_h265 ! videoconvert ! videoscale ! video/x-raw,width=1280,height=720,format=BGRA ! autovideosink sync=false
```

В "классическом режиме" (на камере стримит Majestic, а NVR пересылает видеопоток на ПК, не забываем указать IP адрес ПК в wfb.conf рег故事）视频包括QGroundControl、任务规划器和任务规划器以及其他应用程序。 Для этого нужно установить [GStreamer](https://gstreamer.freedesktop.org/download/) 和 запускать его на прием с некими параметрами, например: В данном приме该视频分辨率为 1280x720。 Для запуска видео с разрешением оригинального потока убираем из строки `videoscale！ ` 和 `width=1280,height=720,`。


![预览]（https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/Screenshot_2.png）

Для воспроизведения видео в окне Mission Planner нужно кликнуть правой кнопкой мыши по его окну с горизонтом и выбрать`视频 > 设置 GStreamer 源`, сти строку параметров: `udpsrc port=5600 buffer-size=32768 !应用程序/x-rtp！ rtph265depay！队列最大大小缓冲区=5！ avdec_h265 ！视频转换！视频/x-raw，格式=BGRA！ appsink name=outsink`, 好的。 Строка сохранится для будущий применений。


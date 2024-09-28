## Добавляем плавности видео На режимах 1080p@30fps заметно легкое подергивание видео, а при замедленной съемке таймера和 картинка замирает на какое то время 和 далее обновляется。 Это происходит из за неравномерности потока, который резко возрастает на ключевых кадрах。 Исправить это можно, перенастроив на камере два параметра энкодера. Спасибо за проделанную работу TipoMan 和 widgetii！

Нам нужно положить файл [gkrcparams](https://github.com/OpenIPC/sandbox-fpv/raw/master/user_TipoMan/gkrcparams) в /usr/sbin, дать права на выполнение `chmod +x /usr/sbin/gkrcparams ` 和 /etc/init.d/S95majestic 中的 Majestic：

```
	start-stop-daemon -b -m -S -q -p "$PIDFILE" -x "/usr/bin/$DAEMON" -- $DAEMON_ARGS
	sleep 1                        <=== ЭТО ВСТАВИТЬ
	gkrcparams --MaxQp 30 --MaxI 2 <=== ЭТО ВСТАВИТЬ
	status=$?
```
После перезапуска картинка должна стать плавной。 majestic.yaml 中的内容位于 mcs1 中：

```
video0:
  enabled: true
  bitrate: 7168
  codec: h265
  rcMode: cbr
  gopSize: 1.0
  size: 1920x1080
```

Если же картинка все равно иногда подергивается, придется изменить mcs на 3 в `/etc/wfb.conf` потеря в дальности уменьша ть битрейт。


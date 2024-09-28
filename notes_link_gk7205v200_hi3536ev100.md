
### Заметки по постройке видеолинка с двухсторонней телеметрией mavlink

Воздушная часть линка представляет из себя камеру [gk7205v200](https://sl.aliexpress.ru/p?key=e1sTwWg) с подключенным по USB wifi-адаптером на以及 rtl8812au，例如 ASUS USB AC-56 以及 [недорогого более слабого адаптера с ali](https://sl.aliexpress.ru/p?key=8CsTwDB)。 Наземная часть - это [видеорегистратор](https://sl.aliexpress.ru/p?key=L1sTwWG) на базе чипа hisilicon hi3536dv100 либо ev100, к которому точно так该产品是 USB 接口的 RTL8812AU 和 RTL8814AU。 Для РФ дешевле 和 быстрее заказать камеру 和 регистратор у [@ser177](https://t.me/ser177)。 Данная статья описывает нюансы создания подобного линка, и является дополнением к [этой статье](https://github.com/OpenIPC/wiki/blob/master/ru/fpv.m d).

![link_hw](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/link_hw.png)

### 胜利者

Данный линк способен передавать видео [(youtube)](https://youtu.be/ldfQ9CLE86I) с воздушной части форматом (разрешение@частота кадров) 19 20x1080@30 和 1280x720@50 和 h264 和 h265 和 mavlink 都适用。 Общая схема процессов для передачи видео выглядит так:

![视频](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/video.png)

视频中的内容是 момент возможна двумя способами。 Классический - 流媒体`Majestic` через udp порт 5600 засылает RTP/h264 или RTP/h265 поток в [wfb-ng](https://github.com/svpcom/wfb-ng) или [OpenHD-wfb](https:/ /github.com/OpenHD/wifibroadcast) 在 землю 上使用，在 PC 上使用 wfb 和 отправляется воспроизведения埃特 /包括 LAN 和 USB 以太网（网络共享）。 Формат RTP свободно воспроизводится программами для GS, такими как [QGroundControl](https://github.com/mavlink/qgroundcontrol), [Mission Planner](https://ardupilot.org/planner/), [QOpenHD](https: //openhdfpv.org/download/) 和 [FPV-VR](https://github.com/Consti10/FPV_VR_OS)。与 HDMI 解决方案相关的技术，包括 специализированном 和 обычн GStreamer 是在 Raspberry Pi 上使用的，适用于 Raspberry Pi。

<sub>QGroundControl 是一款 H265 的 QGroundControl，它可以在 зависании картинки 和 меню 中使用。 Это лечится до перезапуска программы выбором потока видео h264 和 назад h265.</sub>

Andrey Bezborodov 和 OpenIPC предоставил на тесты скомпилированные примеры *vencoder* 和 *vdecoder*, вытащенные из Hisilicon SDK 和 оригинальном виде сположенные [тут](https://github.com/OpenIPC/silicon_research)。 `venc` запускается на камере и формирует поток h264 с фрагментацией HAL вместо `majestic`, `vdec` на регистраторе выводит эток HDMI。屏幕上显示了 OSD 和 OSD 信息。 Это очень перспективный путь，поскольку имеет возможности к снижению задержки передачи видео。时间为 110 至 130 米。 На "классической схеме" задержки составляют обычно от 150 до 230 мсек, [вот пример 133 мсек](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/ Screenshot_1.png), в зависимости от разрешения和 наземных условий воспроизведения。

Этот вопрос можно решить, "научив" `vdec` распознавать RTP/h26[4-5] с помощью библиотеки *libavformat/avformat.h* 和 по прежнему "стримить" на камере через “雄伟”。 Для этого нужна помощь программиста C++，если вы желаете помочь проекту с этим - [обращайтесь](https://t.me/+BMyMoolVOpkzNWUy)。

*GStreamer* 上的 Mission Planner h265: `udpsrc port=5600 buffer-size=32768 !应用程序/x-rtp！ rtph265depay！ avdec_h265 ！视频转换！视频/x-raw，格式=BGRA！ appsink名称=outsink`

适用于 5,2ghz 和 5,85ghz，上 atheros 2.3 - 2.4ghz。

### Как все запускается？

运行 linux 系统，并使用 `init.d` 和 `S98datalink`，以及 является отправной точкой。 Он запускает скрипт `/usr/bin/wifibroadcast`, который определяет через lsusb какой адаптер подключен, загружает его драйвер, т в режим монитора, стартует `wfb` на передачу или прием, для наземки определя подключения по usb, второму адаптеру wifi или与`udp_addr` 相同。 Данные о настройках он берет из `/etc/wfb.conf`。 Также, при включенной телеметрии, он запускает скрипт `/usr/bin/telemetry`, который занимается тем же но для телеметрийныхй целе , беря настройки из `/etc/telemetry.conf`。

### Нюансы на камере

Для данной камеры существует два драйвера сенсора - “медленный” 1080p@30fps 和 “быстрый” 720p@50fps。 Их можно переключать на ходу скриптами из примеров в [root](https://github.com/OpenIPC/sandbox-fpv/tree/master/gk7205v200/root), если залить на камеру ["быс трый" драйвер](gk7205v200/lib /sensors/libsns_imx307_2l_720p.so) под отдельным именем 和 исправить к нему путь в его конфиге [`imx307_i2c_2l_720p_50fps.ini`](gk7205v200/etc/sensors/imx307_i2c_2l_720p_50fps.ini#L15)。 Все файлы по данной камере находятся в каталоге `gk7205v200`。 Если запускать камеру с "быстрым" драйвером в настройках Majestic, то передача видео идет рывками, поэтому при старте камеры S95goke 的型号为“медленного”драйвера，после чего уже можно включить“быстрый”。 На текущий момент [ведется работа](notes_cam_control.md) по управлению подобными настройками камеры через RC каналы в mavlink。

### Нюансы на регистраторе Так как регистратор относительно камеры имеет шикарные 16б памяти spi flash, из которых мы можем использовать около 5мб, то нам доступен [драйвер адаптеров RTL](https://github.com/OpenIPC/sandbox-fpv/tree /master/hi3536dv100/88XXau-ko) который поддерживает rtl8814au 是 rtl8812au 的缩写。 Для этого нужно залить его поверх штатат​​ного в `lib/modules/4.9.37/extra`, не забыв переименовать.

Перекомпилирован [`mavlink-router`](https://github.com/OpenIPC/sandbox-fpv/tree/master/hi3536dv100/usr/bin), так как комплектный из прошивки собран на musl для водзу шной части (где он не используется) ），一个 glibc 上的程序。

Также необходимо [отключить hisilicon watchdog](note_nvr_wdt.md)。

### Нюансы телеметрии Текущая схема работы телеметрии выглядит так:

![遥测]（https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/telemetry.png）

~~С применением mavlink-routerd на текущий момент возможна только односторонняя телеметрия по udp, поскольку он не умеет использовать разные rx/tx udp порты в рамках одного endpoint, как того требует wfb, будучи запущенным разными процессами `telemetry_rx` и `telemetry_tx`.~~
Будучи запущенным разными процессами, `telemetry_rx` и `telemetry_tx` используют разные порты для приема и передачи данных (кстати, это просто символьные ссылки на wfb_rx и wfb_tx, создаваемые скриптом [запуска телеметрии](hi3536dv100/usr/bin/telemetry)), и mavlink-router требует в [конфигурации](hi3536dv100/etc/mavlink.conf) два UDP-endpoint, которые должны быть сгуппированны:
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

~~С применением mavlink-routerd 为 udp、поскольку он не умеет ь rx/tx udp порты в рамках одного 端点，как того требует wfb, будучи запущенным разными процессами `telemetry_rx` 和 `telemetry_tx`。 ~~ Будучи запущенным разными процессами、`telemetry_rx` 和 `telemetry_tx` используют разные порты для приема 和 передачи данных (кстати, это просто символьные ссылки на wfb_rx 和 wfb_tx, создаваемые скриптом [запуска телеметрии](hi3536dv100/usr/bin/telemetry)), 和mavlink-路由器требует в [конфигурациии](hi3536dv100/etc/mavlink.conf) два UDP-endpoint, которые должны быть сгуппированны: Остальные 端点 нужны для связи с前身为 tcp:5760，前身为 Mission Planner。 Для [приложенного конфига](hi3536dv100/etc/mavlink.conf) в настройках UDP-линка нужно указать адрес регистратора:

![udp-qgc](notes_files/qgc-udp-settings.png)

在 /usr/bin/telemetry 中安装了 mavlink-routerd 和 uart 解决方案。

```
  /usr/bin/mavlink-routerd -c /etc/mavlink.conf &
  #/usr/sbin/mavfwd --master ${serial} --baudrate ${baud} --out 127.0.0.1:${port_tx} --in 127.0.0.1:${port_rx} &
```

Если же вы хотите использовать uart, то можете настроить на /dev/ttyAMA0 или переключиться на mavfwd.如果您在 /etc/inittab 中使用 ssh 或 uart，则可以使用以下命令：

```
#console::respawn:/sbin/getty -L  console 0 vt100 # GENERIC_SERIAL
```
Тогда телеметрия станет доступна на UART регистратора взамен или дополнительно к udp по сети, и ее можно будет использоват USB-UART 是串行端口。 QGC 提供了一系列先进的流量控制功能，包括高级流量控制、高级流量控制和高级流量控制。

Я скомпилировал `mavfwd` для регистратора, поддерживающий скорости b230400, b500000, b921600 和 b1500000, для поддержки е высокой скорости при работе с Mission Planner, проверил на b500000 при b115200 / b230400 на камере. Для регистратора его можно забрать [здесь](hi3536dv100/usr/sbin)。 Для камеры: [здесь](https://github.com/OpenIPC/sandbox-fpv/tree/master/gk7205v200/usr/sbin)。 [Исходник](https://github.com/OpenIPC/sandbox-fpv/tree/master/mavfwd)。 На камере удалось получить устойчивую связь с полетником на скорости 230400，выше stm32f4 не смог。 Установка скорости полетника под ardupilot производится параметром `SERIALx_BAUD`，请输入：“230”。 Также не забывайте установить параметр `TELEM_DELAY` на 10 (секунд задержки перед началом выдачи телеметрии), иначе я может остановит загрузчик。 К сожалению, если в полете камера перезапустится по какой то причине отдельно от полетника, то телеметрия ~~не даст ей агрузиться~~ может не дать ей загрузиться. ~~Необходимо доработать загрузчик u-boot, чтобы он не останавливал загрузку по любому символу.~~ ~~C [новым u-boot](gk7205v200_u-boot-7 502v200-for-telemetry.md),~~ который прерывается только по Ctrl +C，并`bootdelay=0` этой проблемы по тестам нет。该 U-boot 可以支持 FPV 和 OpenIPC。

Также я [заложил в него](notes_cam_control.md) основы для наблюдения за выбранными каналами RC mavlink и передачи их значений при ``/root/channels.sh` 相当于 $1 (канал) 和 $2 (значение)。

### Текущие проблемы
Если при загрузке камеры в majestic выбран "быстрый" драйвер 720p, то видео идёт рывками, поэтому в S95goke (автозапуск majestic) перед стартом идёт установка "обычного" драйвера 1080p. Если вы хотите использовать при загрузке камеры 720p@50 по умолчанию, вставьте после загрузки majestic вызов скрипта переключения в [`/etc/init.d/S95majestic`](gk7205v200/etc/init.d/S95majestic#L35) в функции `load_majestic`:
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


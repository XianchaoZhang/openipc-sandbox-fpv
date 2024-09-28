## Настройка кнопок со своим функционалом на регистраторе

![前面板](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/IMG_20230323_081622_212.jpg)

Регистратор имеет на борту разъем для подключения фронт-панели с кнопками, ir-ресивером:

![nvr 端口](notes_files/photo_2023-03-23_02-12-40.jpg)

Разъем cn5 подписан с обратной стороны. С назначением +3.3в и GND понятно, IR задействовать не удалось, остальные пины ведут на GPIO процессора:
```
Y2    ^17
Y1    ^6
X2    ^13
Y3    ^8
X1    ^7
ALARM 10
REC   11
```

Разъем cn5 подписан с обратной стороны。 С назначением +3.3в 和 GND понятно, IR задействовать не удалось, остальные пины ведут на GPIO процессора: Символ `^` означает подт яжку резистором к +3.3, значит кнопкой эти пины надо замыкать на GND 和 ловить значение 0. Это реализовано в файле [`root/gpio_monitor.sh`](hi3536dv100/root/gpio_monitor.sh)。 По замыканию пина Y1 на землю он производит рестарт сервиса [wfb](hi3536dv100/etc/init.d/S98wfb), который следом рестартует [теле метрию](hi3536dv100/usr/bin/telemetry), для более удобного подключения смартфона или планшета [по] USB](usb-tethering.md), 或 после смены wifi-адаптера。该命令的作用是“tail -f /tmp/gpio.log”。 GPIO 上的 выход можно посмотреть в [`testgpio.sh`](hi3536dv100/root/testgpio.sh)，以及 можно подключить пин ALARM REC к алмощному светодиоду с резистором для индикациии процессов, например перезапуска wfb-ng как сделано в` gpio_monitor.sh`。

Для запуска монитора как системного демона создадим файл [`/etc/init.d/S99gpio_monitor`](hi3536dv100/etc/init.d/S99gpio_monitor) откуда и будем запускать наш [`root/gpio_monitor.sh`](hi3536dv100/root/gpio_monitor.sh):
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

Для запуска монитора как системного демона создадим файл [`/etc/init.d/S99gpio_monitor`](hi3536dv100/etc/init.d/S99gpio_monitor) откуда и м запускать наш [`root/gpio_monitor.sh`](hi3536dv100/root/ gpio_monitor.sh): 禁用 wifi 和 USB 调制解调器、禁用和禁用загрузки и убеждаемся, что по нажатию кнопки (хотя бы полсекунды подержите) сервисы запускаются. Список запущенных процессов всегда можно посмотреть командой `ps axww`。


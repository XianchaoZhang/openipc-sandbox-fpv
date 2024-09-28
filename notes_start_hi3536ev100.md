### OpenIPC 上用于 FPV 用途的 NVR hi3536ev100 固件注意事项 [EN](en_notes_start_hi3536ev100.md)

本文与固件无关，请使用 https://github.com/OpenIPC/wiki/blob/master/en/fpv-nvr.md，同一篇文章在某些方面可能有用。


<details> <summary>内存如何工作</summary> 首先，您需要弄清楚录音机（以及相机）的内存如何工作以及需要闪存什么。数据以mtd块的形式存储在spi-flash 16mb上：

```
cat /proc/cmdline
mem=150M console=ttyAMA0,115200 panic=20 root=/dev/mtdblock3 rootfstype=squashfs init=/init mtdparts=hi_sfc:256k(boot),64k(env),2048k(kernel),8192k(rootfs),-(rootfs_data)
ls /dev/mtdb*
/dev/mtdblock0  /dev/mtdblock1  /dev/mtdblock2  /dev/mtdblock3  /dev/mtdblock4
```
Как следует из вывода, нулевой блок это загрузчик u-boot; далее идет блок для хранения переменных окружения (`printenv`, `setenv` команды пишут в ОЗУ, а `saveenv` сохраняет именно в этот блок); следом ядро uImage; потом rootfs.squashfs (неизменяемый образ файловой системы); и наконец rootfs_data или он же overlay - изменяемая часть, куда пишутся отличия от rootfs если вы изменяете какие-либо файлы. Таким образом, очистив overlay, мы "скинем" файловую систему до "дефолта":
```
sf probe 0 #выбираем устройство
sf erase 0xA50000 0x500000 #производим очистку
reset #перезагрузка
```
从输出来看，第 0 块是 u-boot 引导加载程序；接下来是存储环境变量的块（`printenv`、`setenv` 命令写入 RAM，`saveenv` 将其保存在该块中）；接下来是uImage核心；然后 rootfs.squashfs （不可变文件系统映像）；最后是 rootfs_data 或覆盖 - 一个可更改的部分，如果您更改任何文件，则会写入与 rootfs 的差异。因此，通过清除覆盖，我们将文件系统“重置”为“默认”：使用“firstboot”命令将固件重置为“出厂”更加容易。

命令的地址计算器可在[此处](https://openipc.org/tools/firmware-partitions-calculation)获得。在我们的例子中，rootfs 分区：8192kB，这意味着覆盖层的起始地址将为 0xA50000。对于8mB的闪光灯和5120kB的rootfs大小的相机，地址会有所不同，包括环境变量！ </详情>

Загрузчик у этого регистратора не имеет пароля, и в него можно попасть через uart/115200 бод, нажав при старте несколько раз Ctrl+C будучи подключенным к порту debug-uart регистратора через адаптер usb-uart 3v3 (ftdi, ch340). Debug uart расположен напротив разъема VGA на противоположном краю платы и подписан как gnd/tx/rx. Загрузчик нам прошивать не требуется, burn не нужен. ENV (переменные окружения) у нас отличаются от заводских, но их проще установить прямо из загрузчика построчно:
```
setenv ipaddr '192.168.0.222' #тут ip в  вашей подсети из свободных
setenv serverip '192.168.0.107' #адрес ПК с tftp сервером
setenv netmask '255.255.255.0'
setenv bootcmd 'sf probe 0; sf read 0x82000000 0x50000 0x200000; bootm 0x82000000'
setenv uk 'mw.b 0x82000000 ff 1000000;tftp 0x82000000 uImage.${soc}; sf probe 0; sf erase 0x50000 0x200000; sf write 0x82000000 0x50000 ${filesize}'
setenv ur 'mw.b 0x82000000 ff 1000000;tftp 0x82000000 rootfs.squashfs.${soc}; sf probe 0; sf erase 0x250000 0x800000; sf write 0x82000000 0x250000 ${filesize}'
setenv bootargs 'mem=192M console=ttyAMA0,115200 panic=20 root=/dev/mtdblock3 rootfstype=squashfs init=/init mtdparts=hi_sfc:256k(boot),64k(env),2048k(kernel),8192k(rootfs),-(rootfs_data)'
setenv osmem '192M'
setenv totalmem '256M'
setenv soc 'hi3536dv100'
#тут очищаем ненужные далее переменные
setenv da; setenv du; setenv dr; setenv dw; setenv dl; setenv dc; setenv up; setenv tk; setenv dd; setenv de; setenv jpeg_addr; setenv jpeg_size; setenv vobuf; setenv loadlogo; setenv appVideoStandard; setenv appSystemLanguage; setenv appCloudExAbility
saveenv #сохраняем новое окружение переменных
printenv #смотрим, все ли в порядке
```
该记录仪的引导加载程序没有密码，您可以通过 uart/115200 波特率访问它，方法是在启动时按 Ctrl+C 几次，同时通过 usb-uart 3v3 适配器（ftdi）连接到记录仪的 debug-uart 端口，第 340 章）。调试 uart 位于板另一边 VGA 连接器对面，标记为 gnd/tx/rx。我们不需要刷新引导加载程序，也不需要刻录。我们的 ENV（环境变量）与工厂的不同，但它们更容易直接从引导加载程序逐行安装：原始 env 和芯片的完整转储（工厂固件的 16mb 备份，以备恢复）可用[此处](https://github.com/OpenIPC/sandbox-fpv/tree/master/hi3536dv100/original_firmware)。

您可能已经注意到， uk 和 ur 变量存储用于刷新 uImage 和 rootfs 的宏，并从 serverip 变量中指定的 [tftp 服务器](https://pjo2.github.io/tftpd64/) 下载它们。所有地址都对应于 bootargs 变量，其内容指定内核在启动时的文件系统布局。布局与通常的国科/海思相机不同，我们的核心与lite/fpv相同，大小为2MB，但文件系统大小为8MB，与Ultimate一样。剩余的约 5MB 由覆盖层使用（您对相对于原始 rootfs 的文件进行的更改）。对于固件，请使用发布页面 [openipc/firmware](https://github.com/OpenIPC/firmware/releases/download/latest/openipc.hi3536dv100-nor-fpv.tgz) 中的官方版本。该存档包含内核和文件系统。

所以，设置完变量后，就可以开始刷写剩下的部分了。启动tftpd服务器，将uImage.hi3536dv100和rootfs.squashfs.hi3536dv100放入其根目录中，选择适当的网络接口并在引导加载程序中运行宏：`run uk`。必须执行一系列命令，其输出应表明 uImage 文件已下载并刷新到闪存中。同样，运行“run ur”来刷新 rootfs。如果地址设置正确，但下载卡在“正在下载”，请将注册商地址更改为附近的免费地址：`setenv ipaddr '192.168.0.223'`。如果一切顺利，没有错误，请执行“重置”并启动进入操作系统，登录 root，密码 12345。

hi3536dv100 目录中的配置不相关，但它们可能对通过 USB/wifi/以太网热点连接平板电脑感兴趣；您可以类比地将它们转移到官方固件的配置或使用单独的 bash 脚本。通常，这些更改的本质是确定所连接平板电脑的地址（在平板电脑具有 dhcp 服务器的情况下，这是注册器的网关），并在用于视频流和遥测的 wfb_rx 的附加实例中指定此地址溪流。

使用命令“sysupgrade -r -k -n”通过互联网更新固件。

<details> <summary>从 /tmp 进行无网络更新</summary> 以后，您可以通过 WinSCP 将内核和 rootfs 上传到 `/tmp` 目录并运行 `sysupgrade -- 来更新刻录机固件内核=/tmp/uImage.hi3536dv100 --rootfs=/tmp/rootfs.squashfs.hi3536dv100 -z`。如果您没有互联网连接（不更新 sysupgrade 脚本），则需要“-z”参数，“-n”将清除用户 fs（覆盖）。 </详情>


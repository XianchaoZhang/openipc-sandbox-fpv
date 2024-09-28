### 有关 OpenIPC 上用于 FPV 目的的 NVR hi3536ev100 固件的说明 [RU](notes_start_hi3536ev100.md)

<details> <summary>内存的工作原理</summary> 首先，您需要弄清楚录像机（以及摄像机）的内存是如何工作的，以及需要刷新什么。数据以 mtd 块的形式存储在 spi-flash 16mb 上：

```
cat /proc/cmdline
mem=150M console=ttyAMA0,115200 panic=20 root=/dev/mtdblock3 rootfstype=squashfs init=/init mtdparts=hi_sfc:256k(boot),64k(env),2048k(kernel),8192k(rootfs),-(rootfs_data)
ls /dev/mtdb*
/dev/mtdblock0  /dev/mtdblock1  /dev/mtdblock2  /dev/mtdblock3  /dev/mtdblock4
```
As follows from the output, block zero is the u-boot bootloader; Next comes a block for storing environment variables (`printenv`, `setenv` commands are written to RAM, and `saveenv` saves it in this block); next is the uImage core; then rootfs.squashfs (immutable file system image); and finally rootfs_data or also overlay - a changeable part where differences from rootfs are written if you change any files. Thus, by clearing the overlay, we will reset the file system to default:
```
sf probe 0 #select a device
sf erase 0xA50000 0x500000 #we clean
reset #reboot nvr
```
从输出结果可以看出，块零是 u-boot 引导程序；接下来是用于存储环境变量的块（“printenv”、“setenv” 命令写入 RAM，而“saveenv” 将其保存在此块中）；接下来是 uImage 核心；然后是 rootfs.squashfs（不可变文件系统映像）；最后是 rootfs_data 或覆盖 - 一个可更改的部分，如果您更改任何文件，则会在其中写入与 rootfs 的差异。因此，通过清除覆盖，我们将文件系统重置为默认值：使用“firstboot”命令将固件重置为出厂默认值更加容易。

命令的地址计算器可从 [此处](https://openipc.org/tools/firmware-partitions-calculation) 获取。在我们的例子中，rootfs 分区为：8192kB，这意味着覆盖层的起始地址将为 0xA50000。对于闪存为 8mB 且 rootfs 大小为 5120kB 的相机，地址将有所不同，包括环境变量！</details>

The bootloader of this recorder does not have a password, and you can access it via uart/115200 baud by pressing Ctrl+C several times at startup while connected to the debug-uart port of the recorder via a usb-uart 3v3 adapter (ftdi, ch340). Debug uart is located opposite the VGA connector on the opposite edge of the board and is labeled gnd/tx/rx. We don't need to flash the bootloader, we don't need burn. Our ENVs (environment variables) are different from the factory ones, but they are easier to install directly from the bootloader line by line:
```
setenv ipaddr '192.168.0.222' #here is the ip in your subnet from the free ones
setenv serverip '192.168.0.107' #PC address with tftp server
setenv netmask '255.255.255.0'
setenv bootcmd 'sf probe 0; sf read 0x82000000 0x50000 0x200000; bootm 0x82000000'
setenv uk 'mw.b 0x82000000 ff 1000000;tftp 0x82000000 uImage.${soc}; sf probe 0; sf erase 0x50000 0x200000; sf write 0x82000000 0x50000 ${filesize}'
setenv ur 'mw.b 0x82000000 ff 1000000;tftp 0x82000000 rootfs.squashfs.${soc}; sf probe 0; sf erase 0x250000 0x800000; sf write 0x82000000 0x250000 ${filesize}'
setenv bootargs 'mem=192M console=ttyAMA0,115200 panic=20 root=/dev/mtdblock3 rootfstype=squashfs init=/init mtdparts=hi_sfc:256k(boot),64k(env),2048k(kernel),8192k(rootfs) ,-(rootfs_data)'
setenv osmem '192M'
setenv totalmem '256M'
setenv soc 'hi3536dv100'
#here we clear variables that are no longer needed
setenvda; setenv du; setenv dr; setenv dw; setenv dl; setenv dc; setenv up; setenv tk; setenvdd; setenv de; setenv jpeg_addr; setenv jpeg_size; setenv vobuf; setenv loadlogo; setenv appVideoStandard; setenv appSystemLanguage; setenv appCloudExAbility
saveenv #save the new variable environment
printenv #see if everything is ok
```
该记录器的引导加载程序没有密码，您可以在启动时多次按 Ctrl+C，通过 uart/115200 波特率访问它，同时通过 usb-uart 3v3 适配器（ftdi、ch340）连接到记录器的调试 uart 端口。调试 uart 位于电路板另一边的 VGA 连接器对面，标记为 gnd/tx/rx。我们不需要刷新引导加载程序，也不需要刻录。我们的 ENV（环境变量）与出厂环境变量不同，但直接从引导加载程序逐行安装更容易：芯片的原始环境和完整转储（恢复时出厂固件的 16mb 备份）可从[此处]（https://github.com/OpenIPC/sandbox-fpv/tree/master/hi3536dv100/original_firmware）获得。

您可能已经注意到，uk 和 ur 变量存储了 uImage 和 rootfs 的宏，它们从 serverip 变量中指定的 [tftp 服务器](https://pjo2.github.io/tftpd64/) 下载。所有地址都与 bootargs 变量相对应，该变量的内容指定了启动时内核的文件系统布局。该布局与 goke/hisilicone 相机的布局不同，我们的核心与 lite/fpv 相同，大小为 2MB，但文件系统大小为 8MB，与 ultimate 一样。剩余的 ~5MB 由覆盖层使用（您对相对于原始 rootfs 的文件所做的更改）。对于固件，请使用发布页面 [openipc/firmware](https://github.com/OpenIPC/firmware/releases/download/latest/openipc.hi3536dv100-nor-fpv.tgz) 中的官方版本。存档包含内核和文件系统。

因此，设置变量后，您可以开始刷新剩余部分。启动 tftpd 服务器，将 uImage.hi3536dv100 和 rootfs.squashfs.hi3536dv100 放入其根目录中，选择适当的网络接口并运行引导加载程序中的宏：“run uk”。必须执行一系列命令，其输出应表明 uImage 文件已下载并刷新到闪存中。同样，运行“run ur”来刷新 rootfs。如果地址设置正确，但下载停留在“正在下载”状态，请将注册器地址更改为附近的空闲地址：“setenv ipaddr '192.168.0.223'”。如果一切顺利，请执行“重置”并启动操作系统，登录 root，密码为 12345。

hi3536dv100 目录中的配置并不相关，但它们可能与通过 usb/wifi/以太网热点连接平板电脑有关；您可以类似地将它们转移到官方固件的配置中，或者使用单独的 bash 脚本。通常，这些更改的本质是确定连接的平板电脑的地址（如果平板电脑有 dhcp 服务器，则为注册器的网关），并在 wfb_rx 的附加实例中为视频流和遥测流指定此地址。

使用命令“sysupgrade -r -k -n”通过互联网更新固件。

<details> <summary>无需互联网即可从 /tmp 更新</summary> 将来，您可以通过 WinSCP 将内核和 rootfs 上传到 `/tmp` 目录并运行 `sysupgrade --kernel=/tmp/uImage.hi3536dv100 --rootfs=/tmp/rootfs.squashfs.hi3536dv100 -z` 来更新刻录机的固件。如果您没有互联网连接（不更新 sysupgrade 脚本），则需要 `-z` 参数，`-n` 将清除用户 fs（覆盖）。 </details>


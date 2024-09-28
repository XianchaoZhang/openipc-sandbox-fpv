## 使用 OpenIPC 固件刷新 IVG-G2S 相机的注意事项。

注意！本文中的大部分信息已经无可救药地过时了，除了固件之外的所有其他步骤都不再相关 - 它们包含在固件中，并且一切都开箱即用，您只需将 gs.key 从相机复制到地面站并在两者上指定正确的频道。


本说明的目的是帮助引起人们对在没有单板的情况下在廉价相机上运行数字第一人称视角这一极具前景的主题的关注。评论对于开发者来说**非常**重要；如果受欢迎，项目在他们的帮助下会发展得更快。转到专门为此主题创建的 [电报聊天] (https://t.me/+BMyMoolVOpkzNWUy)，提出此处和下面的链接中未描述的问题，发布有关您的成功的信息并寻求解决问题的帮助。再次强调 - 反馈非常重要，即使一切都很顺利并且正在发挥作用 - 请花时间描述你的设置并在聊天中发表评论。这将使项目在开发商的参与下发展。

* [OpenIPC + FPV 主题主页，必读](https://github.com/OpenIPC/wiki/blob/master/ru/fpv.md)
* [项目主要wiki](https://github.com/OpenIPC/wiki)（目前EN版本包含更多信息）
* [网站上的指令生成器](https://openipc.org/supported-hardware/featured)


### 条款

* 闪存 - 在本文中指的是 SPI 闪存、存储芯片。
* U-Boot - 引导加载程序。有一种是"本地"的，有一种来自 OpenIPC。本机受密码保护。我们需要从指令生成器下载一个。
* uImage - 嵌入式Linux内核，以bin文件的形式。
* Root-FS - 所选版本的文件系统（lite、ultimate、fpv），采用 squash-fs 文件的形式 [https://ru.wikipedia.org/wiki/Squashfs]。 uImage和rootfs需要通过指令设计器获得，单独的u-boot和root-fs固件。
* Shell - Linux 相机命令行，可通过 uart 和 ssh（putty 程序）访问。还有一个引导加载程序 shell，仅限 uart。登录是root，没有密码。
* Majestic - 实用程序 - 视频流传输器，来自 OpenIPC 固件套件。

主板通常配备 8 或 16 MB 大小的 spi-flash。终极版本需要16个，可以重新焊接。对于 fpv 目的 8mb 就足够了。

如果您没有以太网电缆将摄像机连接到交换机/路由器，您可以用一半普通跳线和一个 jst xh1.25 8 针连接器制作一根以太网电缆。这些被用于 Omnibus F4 航班和许多其他地方。相机和连接器的引脚很容易通过谷歌搜索；您需要在电缆和相机上连接 4 根线：rx+、rx-、tx+、tx-。同一连接器向引脚 7(gnd) 和 8(vcc) 提供 5 至 12V 的电源。

我建议首先刷新 lite，因为它有一个通过 uart（在 /etc/inittab 文件中）解锁的 shell，这将允许您在紧急情况下进行连接，例如，当没有网络时。

在 OpenIPC 上烧写 ivg-g2s 摄像头的方式有以下三种。按复杂程度排列：coupler、burn、编程器。

- Coupler [https://github.com/OpenIPC/ Coupler] - 这是通过相机的本机 Web 界面将内核和 rootfs 加载到一个文件中。缺点 - 本机受密码保护的引导加载程序不会更改，只能执行一次，回滚或更改固件只能通过其他方法。算法：找出本机Web界面中的版本，根据它下载固件，将其作为本机Web界面中的更新下载。对于 ivg-g2s/659A7，有[带有引导加载程序的固件](gk7205v200/659A7_OpenIPC_FPV.bin)。

- burn[https://github.com/OpenIPC/burn] - 通过摄像头 uart 和 uart-usb 适配器（例如 ch340）从 OpenIPC 加载无密码保护的 u-boot 引导加载程序，并在引导加载程序中进行进一步的工作，根据 OpenIPC 指令设计者的说法。没有缺点，引导加载程序可以更改，您可以随时从 tftpf 服务器 [https://pjo2.github.io/tftpd64/] 下载所需的映像并将其闪存到闪存中。 OpenIPC 频道 [https://www.youtube.com/@openipc/videos] 上的视频说明中描述了该算法。接下来，将引导加载程序加载到 RAM 中（由于细微差别，仅[选择您的](gk7205v200_u-boot-7502v200-for-telemetry.md)），我们通过指令构造函数进行工作。解压uImage和root-fs并将它们放入tftpd目录中。我尝试将usb-uart转发到virtualbox下的archlinux并从那里使用burn，但下载不起作用，与uart的连接是单向的，只读的。 Windows 7 下一切顺利。从 USB 端口到适配器以及从适配器到相机都使用短线。相机上 uart 的引脚排列在上面列表中的第一篇文章中；您需要将其"交叉"连接 - 将 tx 适配器连接到 rx 相机，将 rx 适配器连接到 tx 相机。如果一切正常，但 tftp 无法下载文件，请尝试将摄像头地址更改为附近的免费地址"setenv ipaddr '192.168.0.223"。

- 编程器 - 拆焊闪存或一条腿并连接到编程器，通过编程器烧写所有内容。最难的方法，这是唯一的缺点。

首次启动时，必须运行firstboot命令。如果摄像机周期性重启，则会触发 Majestic（streamer）的看门狗。您很可能需要取下镜头盖并打开灯。在 /etc/majestic.yaml 中禁用，快速终止进程：`killall majestic`。

您可以通过 shell cli 实用程序通过保存来管理配置参数：
```
cli -s .image.contrast 50
cli -s .image.luminance 50
cli -s .video0.codec h264
cli -s .hls.enabled false
cli -s .isp.sensorConfig /etc/sensors/imx307_i2c_2l_1080p.ini
cli -s .video0.size 1920x1080
cli -s .video0.fps 30
```
我建议运行这些命令，这将为初次尝试配置 Majestic 奠定基础。

转到有更新的 FPV 版本。注意！在 FPV 版本中，通过 uart 的 shell 被禁用（释放用于遥测工作），加载后仅保留网络访问权限。当然，引导加载程序本身是通过 uart 工作的。
```
    sed -i 's/BUILD_OPTION=lite/BUILD_OPTION=fpv/' /etc/os-release
    sysupgrade --force_ver -k -r -n
```
如果您不打算通过此视频链接使用遥测，则可以将其返回：
```
     sed -i 's/#console::respawn:\/sbin\/getty/console::respawn:\/sbin\/getty/' /etc/inittab
     reboot
```
如果闪存被锁定（您以某种方式刷新了无法解锁闪存的早期版本的内核），则不会执行此命令和任何其他命令。

 `dmesg | grep bsp-sfc` 确定闪存驱动器是否被锁定：

解锁
```
bsp-sfc bsp_spi_nor.0: SR1:[02]->[00]
bsp-sfc bsp_spi_nor.0: SR2:[02]->[00]
bsp-sfc bsp_spi_nor.0: all blocks are unlocked.
bsp-sfc bsp_spi_nor.0: Winbond: SR1 [], SR2 [QE], SR3 [DRV0,DRV1]
```

锁定
```
bsp-sfc bsp_spi_nor.0: SR1:[02]->[00]
bsp-sfc bsp_spi_nor.0: SR2:[02]->[00]
bsp-sfc bsp_spi_nor.0: all blocks are unlocked.
bsp-sfc bsp_spi_nor.0: Winbond: SR1 [TB], SR2 [QE], SR3 [WPS,DRV0,DRV1]
```

尝试从 shell 解锁闪存（对我没有帮助） 
```
devmem 0x10010024 32 0x06;devmem 0x10010030 32 0;devmem 0x1001003C 32 0x81;devmem 0x14000000 16 0x0000;devmem 0x10010024 32 0x01;devmem 0x10010030 32 0;devmem 0x10010038 32 2;devmem 0x1001003C 32 0xA1
#
sysupgrade --force_ver -k -r -n
```

解锁闪存的路径是通过加载新的内核，由存档器从包含内核和 rootfs 的 tgz 文件中解压，取自指令构造函数。加载后，它将自动解锁闪存。它是由通过烧录加载到 RAM 中的引导加载程序制成的。

```
# 设置摄像头地址和tftpd服务器地址
setenv ipaddr 192.168.0.222; setenv serverip 192.168.0.107

# 设置环境参数 (env)
setenv bootargs 'mem=32M console=ttyAMA0,115200 panic=20 rootfstype=squashfs root=/dev/mtdblock3 init=/init mtdparts=sfc:256k(boot),64k(env),2048k(kernel),5120k(rootfs),-(rootfs_data)'

# 清除用于下载内核的RAM区域
mw.b 0x42000000 ff 1000000

# 从tftpd服务器加载uImage.gk7205v200内核
tftpboot 0x42000000 uImage.${soc}

# 启动内核
bootm 0x42000000
```

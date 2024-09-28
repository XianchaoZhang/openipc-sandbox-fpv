## 通过 LTE (4G) 调制解调器将相机连接到平板电脑或 PC

对于稳定或自动选路的飞机来说，通过 4G 飞行是一个非常有趣的话题。让我们将设置过程分解为任务：

* 通过相机上的调制解调器设置互联网
* 设置您自己的零层服务器（您可以使用公共服务器）
* 将相机和PC连接到同一个zerotier网络并设置流

Следуя [этим инструкциям](usb-modeswitch.md), настроим usb_modeswitch и сетевой интерфейс eth1 на камере под прошивкой OpenIPC LTE. Если у вас FPV или LITE прошивка, предварительно нужно ее сменить онлайн:
```
#тут меняем fpv на lte в файле /etc/os-release, можно это сделать вручную
sed -i 's/BUILD_OPTION=fpv/BUILD_OPTION=lte/' /etc/os-release
#а это если у вас lite версия
sed -i 's/BUILD_OPTION=lite/BUILD_OPTION=lte/' /etc/os-release

sysupgrade --force_ver -k -r -n
```
按照[这些说明](usb-modeswitch.md)，我们将在 OpenIPC LTE 固件下配置相机上的 usb_modeswitch 和 eth1 网络接口。如果您有FPV或LITE固件，首先需要在线更改：我们收到带有出厂设置和lte固件的相机，其中与fpv不同，删除了wfb，而是安装了zerotier-one客户端。事实上，正确的解决方案是不使用usb_modeswitch，而是将调制解调器的辅助组成直接配置为cdc_ethernet。那么调制解调器将不再通用，并且将立即显示为网卡，但许多问题的可能性将消失。

#### zerotier
Это программное обеспечение для объединения нескольких устройств в одну локальную сеть. Существует публичный сервер для создания своей сети, но лучше поднять свой.
Для этого потребуется vps-сервер под ubuntu.
```
apt-get install -y apt-transport-https gnupg mc iftop          #устанавливаем зависимости
curl -s https://install.zerotier.com | sudo bash               #устанавливаем клиентскую часть

curl -O https://s3-us-west-1.amazonaws.com/key-networks/deb/ztncui/1/x86_64/ztncui_0.7.1_amd64.deb #устанавливаем панель управления
apt-get install ./ztncui_0.7.1_amd64.deb

echo 'HTTPS_PORT=6443' > /opt/key-networks/ztncui/.env         #порт для вебморды управления
echo 'NODE_ENV=production' >> /opt/key-networks/ztncui/.env    #режим работы
echo 'HTTPS_HOST=nn.mm.ff.dd' >> /opt/key-networks/ztncui/.env #внешний ip-адрес нашего сервера

systemctl restart ztncui
```

#### Zerotier 这是一款用于将多个设备连接到一个本地网络的软件。有一个公共服务器可以创建您自己的网络，但最好创建您自己的网络。为此，您需要一个适用于 ubuntu 的 vps 服务器。使用链接https://ip_addr:6443登录，登录admin，密码password。接下来，我们创建一个网络并配置发布地址的参数，您最喜欢哪个，其余设置为默认。在私有模式下，连接客户端后，需要勾选授权复选框以允许他连接。

![ZTNCUI](https://github.com/OpenIPC/sandbox-fpv/raw/master/notes_files/ZTNCUI.png)

有一种[公共服务器](https://my.zerotier.com/)形式的替代方案，但可靠性和性能问题仍然不确定。程序 - Windows、Android 客户端下载[此处](https://www.zerotier.com/download/)。

Подключение к сети производится через указание Network ID, 16-значной символьной строки, которую берем из панели управления. Для камеры ее указываем в /etc/datalink.conf
```
use_zt=true
zt_netid=a8867b0bxxxxxxxxx
```
после чего перезагружаем камеру. При наличии интернет-подключения, хоть LTE хоть ethernet, камера должна подключиться к сети zerotier. Это можно проверить через веб-панель управления и на камере командой ifconfig.
```
ztuplek3wb Link encap:Ethernet  HWaddr 92:31:B1:54:8B
          inet addr:10.7.0.1  Bcast:10.7.0.255  Mask:255.255.255.0
          inet6 addr: fe80::9031:b1ff:fe54/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:2800  Metric:1
          RX packets:93 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1236835 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:5677 (5.5 KiB)  TX bytes:1493618333 (1.3 GiB)
```

连接到网络是通过指定网络 ID 来完成的，网络 ID 是我们从控制面板获取的 16 位字符串。对于摄像头，请在 /etc/datalink.conf 中指定它，然后重新启动摄像头。如果您有互联网连接（无论是 LTE 还是以太网），摄像机应连接到 Zerotier 网络。这可以通过网络控制面板和使用 ifconfig 命令在摄像机上进行检查。  对于 PC 或 Android 设备，[安装](https://www.zerotier.com/download/) 程序并类似地通过其 ID 添加网络，在 Web 面板中授权设备。我们尝试交叉 ping，应该可以。如果您有防火墙/防火墙，例如在Windows下，您需要添加一条允许规则，其中包含我们的子网。

#### Настройка стрима
Остается в /etc/majestic.yaml указать ip-адрес наземки из сети zerotier и видео можно принимать. Не забудьте согласовать кодеки.
```
outgoing:
- udp://ip_from_zerotier:5600
```

#### Телеметрия
Проверку телеметрии я еще не делал, но работать все должно как то так.
Используется mavlink-routerd с конфигом /etc/mavlink.conf. Нужно указать эндпоинты для локального serial и наземки по ip-адресу zerotier:
```
[General]
TcpServerPort = 0

[UartEndpoint drone]
Device = /dev/ttyAMA0
Baud = 115200

[UdpEndpoint qgroundcontrol]
Mode = Normal
Address = gs_ip_from_zerotier
Port = 14550
```

#### 设置流 您所要做的就是在 /etc/majestic.yaml 中指定来自 Zerotier 网络的登陆器的 IP 地址，然后就可以接收视频了。不要忘记协商编解码器。
#### 遥测 我还没有检查遥测，但一切都应该像这样工作。使用 mavlink-routerd 和配置 /etc/mavlink.conf。您需要通过 Zerotier IP 地址指定本地串行和地面的端点：由于连接是双向的，我们会自动接收两个方向的遥测数据。


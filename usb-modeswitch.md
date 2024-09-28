### 在带有 fpv、lite 和 NVR 固件 hi3536[ed]v100 的相机上安装 usb_modeswitch

我们检查 e3372h 调制解调器。

Камера:
```
curl -o /usr/sbin/usb_modeswitch http://fpv.openipc.net/files/usb-modeswitch/musl/usb_modeswitch && chmod +x /usr/sbin/usb_modeswitch
curl -o /usr/lib/libusb-1.0.so.0.3.0 http://fpv.openipc.net/files/usb-modeswitch/musl/libusb-1.0.so.0.3.0 && chmod +x /usr/lib/libusb-1.0.so.0.3.0
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so.0
```
NVR:
```
ntpd -Nnq
curl -k -L -o /usr/sbin/usb_modeswitch https://github.com/OpenIPC/sandbox-fpv/raw/master/usb-modeswitch/musl/usb_modeswitch && chmod +x /usr/sbin/usb_modeswitch
curl -k -L -o /usr/lib/libusb-1.0.so.0.3.0 https://github.com/OpenIPC/sandbox-fpv/raw/master/usb-modeswitch/musl/libusb-1.0.so.0.3.0 && chmod +x /usr/lib/libusb-1.0.so.0.3.0
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so.0
ln -s -f /lib/libc-2.32.so /lib/libc.so
```


<details>
  <summary>альтернативное хранилище</summary>
  
```

curl -o /usr/sbin/usb_modeswitch http://fpv.openipc.net/files/usb-modeswitch/glibc/usb_modeswitch && chmod +x /usr/sbin/usb_modeswitch
curl -o /usr/lib/libusb-1.0.so.0.3.0 http://fpv.openipc.net/files/usb-modeswitch/glibc/libusb-1.0.so.0.3.0 && chmod +x /usr/lib/libusb-1.0.so.0.3.0
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so.0
ln -s -f /lib/libc-2.32.so /lib/libc.so
```
摄像头：NVR：<详细信息> <摘要>备用存储</摘要> </详细信息>



Вносим этот текст для e3372h в файл `/etc/network/interfaces.d/eth1` (создадим файл если отсутствует):
```
auto eth1
iface eth1 inet dhcp
    pre-up sleep 4
    pre-up if [ ! -z "`lsusb | grep 12d1:1f01`" ]; then usb_modeswitch -v 0x12d1 -p 0x1f01 -J; fi
    pre-up if [ ! -z "`lsusb | grep 12d1:14dc`" ]; then modprobe usbserial vendor=0x12d1 product=0x14dc; fi
    pre-up modprobe rndis_host
    pre-up sleep 2
```

我们将 e3372h 的此文本添加到文件“/etc/network/interfaces.d/eth1”中（如果该文件丢失，我们将创建该文件）：我们扭曲调制解调器，尝试“ifup eth1”或重新启动。如果网络连接正常（“ip a”中有eth1），在接口中我们可以将手动替换为自动。

#### 问题 如果 usb_modeswitch 将调制解调器切换到 cdc_ethernet，则当系统重新启动（例如通过重新启动）时，接口不会启动 - 错误 ip: SIOCGIFFLAGS: No such device。因此，如果需要完全重新启动系统以使调制解调器正常工作，则需要在重新启动之前切断调制解调器的电源。

### Результат
Модем e3372h с прошивкой hilink должен отобразиться сетевым интерфейсом eth1 и при вставленной работоспособной sim-карте раздавать интернет на камеру:
```
Trying to send message 1 to endpoint 0x01 ...
 OK, message successfully sent
Read the response to message 1 (CSW) ...
 Device seems to have vanished after reading. Good.
 Device is gone, skip any further commands
-> Run lsusb to note any changes. Bye!

udhcpc: started, v1.36.0
udhcpc: broadcasting discover
udhcpc: broadcasting discover
udhcpc: broadcasting discover
udhcpc: broadcasting discover
udhcpc: broadcasting select for 192.168.8.100, server 192.168.8.1
udhcpc: lease of 192.168.8.100 obtained from 192.168.8.1, lease time 86400
deleting routers
adding dns 192.168.8.1
adding dns 192.168.8.1
OK
```
ifconfig:
```
eth1      Link encap:Ethernet  HWaddr 0C:5B:8F:27:9A:64
          inet addr:192.168.8.100  Bcast:192.168.8.255  Mask:255.255.255.0
          inet6 addr: fe80::e5b:8fff:fe27:9a64/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:34 errors:0 dropped:0 overruns:0 frame:0
          TX packets:806 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4557 (4.4 KiB)  TX bytes:822513 (803.2 KiB)

```

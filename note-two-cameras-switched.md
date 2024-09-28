## 一个链接上有多个摄像头

У камер есть сетевой интерфейс, который в случае двух камер можно использовать даже без свича, просто соединив четыре провода интерфейса друг с другом. Этот интерфейс и будем использовать для связи между камерами.
Если /etc/network/interfaces доработать примерно таким образом:
```
auto eth0
iface eth0 inet dhcp
    hwaddress ether $(fw_printenv -n ethaddr || echo 00:24:B8:FF:FF:FF)

auto eth0:1
iface eth0:1 inet static
    address $(fw_printenv -n ipaddr || echo 192.168.1.9)
    netmask 255.255.255.0
```
то у камер появится саб-интерфейс с адресом, прописанным в переменной env `ipaddr` либо, если она пуста, указанным в address. Нам нужно, чтобы у камер были адреса из одной подсети, например пусть это будут 192.168.1.9 и 192.168.1.10 у "первой" и "второй" камер.
Первая - та, на которой расположен линк wfb и wifi-свисток. От второй нужен только поток на дополнительный порт, пусть 5601 на адрес первой камеры.
В случае, если на второй камере стоит openipc, нужно на ней отключить wfb через `daemon=0` в `datalink.conf` и настроить udp поток в majestic.yaml на 192.168.1.9:5601.
Теперь создадим на первой камере демонстрационный скрипт переключения камер `camswitch.sh`:
```
function wfb_restart {
  kill -9 $(pidof wfb_tx)
  . /etc/wfb.conf
  wfb_tx -p ${stream} -u ${udp_port} -K /etc/drone.key -B ${bandwidth} -M ${mcs_index} -S ${stbc} -L ${ldpc} -G ${guard_interval} -k ${fec_k} -n ${fec_n} -T ${fec_timeout} -i ${link_id} ${wlan} &
}

function cam_1 {
  # this is main cam, with wfb_tx
  sed -i 's/udp_port=5601/udp_port=5600/' /etc/wfb.conf
  wfb_restart
}

function cam_2 {
  # set '- udp: cam1ip:5601' in /etc/majestic.yaml on cam2
  sed -i 's/udp_port=5600/udp_port=5601/' /etc/wfb.conf
  wfb_restart
}

cam_$1
```
这些摄像机具有网络接口，在有两个摄像机的情况下，即使没有交换机也可以使用该网络接口，只需将四根接口线相互连接即可。我们将使用此接口进行相机之间的通信。如果 /etc/network/interfaces 大致以这种方式修改：那么摄像机将有一个子接口，其地址在环境变量“ipaddr”中指定，或者如果它为空，则在地址中指定。我们需要摄像机具有来自同一子网的地址，例如，“第一”和“第二”摄像机的地址为 192.168.1.9 和 192.168.1.10。第一个是 wfb 链接和 wifi 哨子所在的位置。从第二个开始，您只需要一个流到另一个端口，让 5601 到第一个摄像机的地址。如果第二个摄像头有 openipc，则需要通过 datalink.conf 中的 daemon=0 禁用 wfb，并将 majestic.yaml 中的 udp 流配置为 192.168.1.9:5601。现在让我们创建一个演示脚本，用于在第一个摄像机上切换摄像机“camswitch.sh”：让我们通过“chmod +x camswitch.sh”赋予它执行权限，现在我们可以通过调用“camswitch.sh 1”或“在摄像机之间切换” camswitch.sh 2 `.该脚本停止 wfb_tx，替换其配置中的 udp_port（主摄像头发送到 5600，第二个摄像头发送到 5601）并再次启动，从而在流之间切换。例如，您可以将脚本调用连接到 [channels.sh](notes_cam_control.md) 并控制某个 RC 通道的切换。


## 用于遥测的 U-Boot
#### 仅适用于 gk7502v200！

开发人员在发行版中包含了引导加载程序，如果您按照 openipc.net 上的说明进行刷新，则无需执行此操作，只需将 bootdelay 设置为 0 即可。

Чтобы поток телеметрии не прерывал загрузку камеры, когда u-boot считает телеметрию "нажатием любой кнопки для входа в консоль загрузчика", нужно залить этот загрузчик
 [(ССЫЛКА)](https://github.com/OpenIPC/sandbox-fpv/raw/master/gk7205v200/u-boot-gk7205v200-universal.bin):
```
# закачайте файл u-boot-gk7205v200-universal.bin в /tmp камеры через winscp ИЛИ выполните команду и УБЕДИТЕСЬ ЧТО ФАЙЛ СКАЧАЛСЯ!!!#curl -o /tmp/u-boot-gk7205v200-universal.bin http://fpv.openipc.net/u-boot-gk7205v200-universal.bin

flashcp -v /tmp/u-boot-gk7205v200-universal.bin /dev/mtd0
```
为了防止当u-boot认为遥测是“按任意按钮进入引导加载程序控制台”时遥测流中断相机的加载，您需要上传此引导加载程序[(LINK)](https://github.com/u-boot)。 com/OpenIPC/sandbox-fpv/raw /master/gk7205v200/u-boot-gk7205v200-universal.bin)：并使用命令“fw_setenv bootdelay 0”将启动延迟设置为零。

第一次刷新固件时，只需使用此文件而不是说明页面上指示的文件。仅适用于gk7502v200！

此引导加载程序被 Ctrl+C 组合键中断，而不是任何组合键。当心！引导加载程序失败 - 您将不得不使用程序员进行焊接和缝纫。


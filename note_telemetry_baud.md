## 关于遥测的 uart 速度不同于 115200 的注意事项 
TipoMan 遇到了一个问题：当以不同于 115200 的速度向 uart 发送遥测数据时，相机冻结。解决方案：在 `/etc/inittab` 中设置所需的速度。

![inittab](notes_files/baud38400.jpg)

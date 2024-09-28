## 禁用记录仪上的看门狗

记录仪上的引导加载程序会启动一条狗，每半小时重新启动一次。如需禁用，请将 [`wdt.ko`](hi3536dv100/lib/wdt.ko) 复制到 `/lib` 并添加到 [`/etc/init.d/S95hisilicon`](hi3536dv100/etc/init.d/S95hisilicon ) 加载模块，立即卸载：
```
    insmod /lib/wdt.ko
    rmmod /lib/wdt.ko
```

重新启动，看门狗不应再触发。


# 磁盘IO高

- ##### 故障场景

磁盘IO高导致应用故障或降级

- ##### 演练目标

是否能监控发现

是否有对应的预案

故障恢复的时长

- ##### 模拟手段

方法1：

通过dd不停的写磁盘

```bash
dd if=/dev/zero of=/data/diskfull.drill  bs=1M count=10240  oflag=dsync
```

方法二：阿里chaosblade

https://github.com/chaosblade-io/chaosblade

方法三：vmware脚本

https://github.com/vmware/mangle/blob/master/mangle-default-plugin/src/main/resources/InjectionScripts/ioburn.sh

# CPU利用率高

- ##### 故障场景

应用因为某种原因导致CPU使用率达到100%，导致应用对外服务能力下降或不可用。

- ##### 演练目标

是否能监控发现

是否有对应的预案

故障恢复的时长

- ##### 模拟手段

模拟cpu变高的手段有很多种，列举常用的几种

方法1.使用yes命令（推荐）

```bash
`yes>/dev/null`
```

这个命令只能使cpu的1个核跑满100%，多核的话就起多个命令。

或使用vmware现成的脚本（推荐）：

https://github.com/vmware/mangle/blob/master/mangle-default-plugin/src/main/resources/InjectionScripts/cpuburn.sh

![image-20200426203656302](https://raw.githubusercontent.com/kakukaops/ChaosExpriment/master/images/image-20200426203656302.png)	

方法2.循环跑一个系统调用

```bash
#!/bin/bash

while true
do
    date>/dev/null
done
```

![image-20200426205106032](https://raw.githubusercontent.com/kakukaops/ChaosExpriment/master/images/image-20200426205106032.png)

方法二和方法二的区别在于方法一是us比较高，方法二是sy比较高，因为方法2的date命令会进行疯狂的系统调用，从而导致sy占比较高。

方法一和方法二对演练目的也没啥影响。

方法3.循环跑ssl测试

```bash
#!/bin/bash

while true
do
    /usr/bin/openssl speed>/dev/null 2>&1
done
```

![image-20200427093602850](https://raw.githubusercontent.com/kakukaops/ChaosExpriment/master/images/image-20200427093602850.png)

方法4.使用stress工具

```bash
yum install stress -y

stress -c 4 -t 120
```

![image-20200427093856268](https://raw.githubusercontent.com/kakukaops/ChaosExpriment/master/images/image-20200427093856268.png)

stress简单用法：

-c  cpu核心数量

-t   压测时长



方法5.使用阿里的chaosblade（推荐）

https://github.com/chaosblade-io




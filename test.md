# TC的高级用法和好用的工具

​        网络类的故障一般使用tc命令，tc全称traffic control，是linux下进行网络流控的工具。

​        如果我们要对模拟的故障进行精确的控制，例如我们模拟网络延迟故障时，如果延迟过大，ssh操作可能不便，这个时候我们希望对除ssh之外的流量模拟延迟，这种需要怎么做？或者我们只想对某个ip或者port进行延迟模拟，又应该怎么用，这些情况下就需要使用到tc的高级用法。

例如如下命令：

```bash
/usr/sbin/tc qdisc add dev eth0 root handle 1a1a: htb default 1
/usr/sbin/tc class add dev eth0 parent 1a1a: classid 1a1a:1 htb rate 32000000.0kbit
/usr/sbin/tc class add dev eth0 parent 1a1a: classid 1a1a:3 htb rate 32000000.0Kbit ceil 32000000.0Kbit
/usr/sbin/tc qdisc add dev eth0 parent 1a1a:3 handle 2f1b: netem delay 200.0ms
/usr/sbin/tc filter add dev eth0 protocol ip parent 1a1a: prio 1 u32 match ip sport 22 0xffff flowid 1a1a:1
/usr/sbin/tc filter add dev eth0 protocol ip parent 1a1a: prio 2 u32 match ip dst 0.0.0.0/0 match ip src 0.0.0.0/0 flowid 1a1a:3
```

这命令就比较复杂了，比如handle是什么意思，1a1a:呢？，htb呢？class呢，filter呢，u32（还有u16，u8）?  0xffff?需要看一些资料才能搞明白。

Traffic Control HOWTO    http://www.tldp.org/HOWTO/Traffic-Control-HOWTO/intro.html

Linux TC(Traffic Control)框架原理解析     https://www.cnblogs.com/yxwkf/p/5424383.html

Linux 下 TC 命令原理及详解    https://blog.csdn.net/pansaky/article/details/88801249 



### 一个可以帮我们写tc命令的非常好用的工具 tcconfig

github   （https://github.com/thombashi/tcconfig）

这个工具封装了tc的复杂用法，我们可以通过相对简单的命令生成tc规则。

帮助文档：https://tcconfig.rtfd.io/

安装：

```bash
sudo pip install tcconfig
#或
curl -sL https://raw.githubusercontent.com/thombashi/tcconfig/master/scripts/installer.sh | sudo bash
```

用法：

tcset：设定tc规则

tcdel:删除tc规则

tcshow:显示tc规则

一般情况下，我们在执行正式模拟前，想看看原始的tc命令，可以用--tc-command或--tc-script方法：

```bash
#使用--tc-command可以看到原始的tc命令

tcset --tc-command --delay 200ms --exclude-src-port 22 eth0

#使用--tc-script可以帮我们生成一个包含tc命令的脚本

tcset --tc-script --delay 200ms --exclude-src-port 22 eth0
```

tcconfig的帮助文档非常详细，用法也非常直观，直接tcset --help就可以明白怎么用。

```bash
#例如我们要在eth0上模拟200ms的延迟，但排除22端口。

tcset  --delay 200ms --exclude-src-port 22 eth0
```



```bash
[root@ecs-b520 ~]# tcset --help
usage: tcset [-h] [-V] [--tc-command | --tc-script] [--debug | --quiet]
             [--debug-query] [--stacktrace] [--import-setting]
             [--overwrite | --change | --add] [--rate BANDWIDTH_RATE]
             [--delay NETWORK_LATENCY] [--delay-distro LATENCY_DISTRO_TIME]
             [--loss PACKET_LOSS_RATE] [--duplicate PACKET_DUPLICATE_RATE]
             [--corrupt CORRUPTION_RATE] [--reordering REORDERING_RATE]
             [--shaping-algo {htb,tbf}] [--iptables]
             [--direction {outgoing,incoming}] [--network DST_NETWORK]
             [--src-network SRC_NETWORK] [--port DST_PORT]
             [--src-port SRC_PORT] [--ipv6]
             [--exclude-dst-network EXCLUDE_DST_NETWORK]
             [--exclude-src-network EXCLUDE_SRC_NETWORK]
             [--exclude-dst-port EXCLUDE_DST_PORT]
             [--exclude-src-port EXCLUDE_SRC_PORT] [--docker]
             [--src-container SRC_CONTAINER] [--dst-container DST_CONTAINER]
             device

positional arguments:
  device                target name: network-interface/config-file (e.g. eth0)

optional arguments:
  -h, --help            show this help message and exit
  -V, --version         show program's version number and exit
  --tc-command          display tc commands to be executed and exit. these
                        commands are not actually executed.
  --tc-script           generate a shell script file that described tc
                        commands. this tc script execution result nearly
                        equivalent with the tcconfig command. the script can
                        be executed without tcconfig package installation.
  --debug               for debug print.
  --quiet               suppress execution log messages.
  --import-setting      import traffic control settings from a configuration
                        file.
  --overwrite           overwrite existing traffic shaping rules.
  --change              change existing traffic shaping rules to the new one.
                        this option is effective to reduce the time between
                        the shaping rule switching compared to --overwrite
                        option. note: just adds a shaping rule if there are no
                        existing shaping rules.
  --add                 add a traffic shaping rule in addition to existing
                        rules.

Debug:
  --debug-query         for debug print.
  --stacktrace          print stack trace for debug information. --debug
                        option required to see the debug print.

Traffic Control Parameters:
  --rate BANDWIDTH_RATE, --bandwidth-rate BANDWIDTH_RATE
                        network bandwidth rate [bit per second]. the minimum
                        bandwidth rate is 8 bps. valid units are either: bps,
                        bit/s, [tT]bps, [tT]bit/s, [gG]bps, [gG]bit/s,
                        [mM]ibps, [mM]ibit/s, [mM]bps, [mM]bit/s, [kK]bps,
                        [kK]bit/s, [kK]ibps, [kK]ibit/s, [gG]ibps, [gG]ibit/s,
                        [tT]ibps, [tT]ibit/s. e.g. tcset eth0 --rate 10Mbps
  --delay NETWORK_LATENCY
                        round trip network delay. the valid range is from 0ms
                        to 60min. valid time units are: d/day/days,
                        ms/msec/msecs/millisecond/milliseconds, h/hour/hours,
                        m/min/mins/minute/minutes, s/sec/secs/second/seconds,
                        us/usec/usecs/microsecond/microseconds. if no unit
                        string found, considered milliseconds as the time
                        unit. (default=0ms)
  --delay-distro LATENCY_DISTRO_TIME
                        distribution of network latency becomes X +- Y (normal
                        distribution). Here X is the value of --delay option
                        and Y is the value of --delay-dist option). network
                        latency distribution is uniform, without this option.
                        valid time units are: d/day/days,
                        ms/msec/msecs/millisecond/milliseconds, h/hour/hours,
                        m/min/mins/minute/minutes, s/sec/secs/second/seconds,
                        us/usec/usecs/microsecond/microseconds. if no unit
                        string found, considered milliseconds as the time
                        unit.
  --loss PACKET_LOSS_RATE
                        round trip packet loss rate [%]. the valid range is
                        from 0 to 100. (default=0)
  --duplicate PACKET_DUPLICATE_RATE
                        round trip packet duplicate rate [%]. the valid range
                        is from 0 to 100. (default=0)
  --corrupt CORRUPTION_RATE
                        packet corruption rate [%]. the valid range is from 0
                        to 100. packet corruption means single bit error at a
                        random offset in the packet. (default=0)
  --reordering REORDERING_RATE
                        packet reordering rate [%]. the valid range is from 0
                        to 100. (default=0)
  --shaping-algo {htb,tbf}
                        shaping algorithm. defaults to htb (recommended).
  --iptables            use iptables to traffic control.

Routing:
  --direction {outgoing,incoming}
                        the direction of network communication that imposes
                        traffic control. 'incoming' requires ifb kernel module
                        and Linux kernel 2.6.20 or later. (default = outgoing)
  --network DST_NETWORK, --dst-network DST_NETWORK
                        specify destination IP-address/network that applies
                        traffic control. defaults to any.
  --src-network SRC_NETWORK
                        specify source IP-address/network that applies traffic
                        control. defaults to any. this option has no effect
                        when executing with "--direction incoming" option.
                        note: this option required to execute with the
                        --iptables option when using tbf algorithm.
  --port DST_PORT, --dst-port DST_PORT
                        specify destination port number that applies traffic
                        control. defaults to any.
  --src-port SRC_PORT   specify source port number that applies traffic
                        control. defaults to any.
  --ipv6                apply traffic control to IPv6 packets rather than
                        IPv4.
  --exclude-dst-network EXCLUDE_DST_NETWORK
                        exclude a specific destination IP-address/network from
                        a shaping rule.
  --exclude-src-network EXCLUDE_SRC_NETWORK
                        exclude a specific source IP-address/network from a
                        shaping rule.
  --exclude-dst-port EXCLUDE_DST_PORT
                        exclude a specific destination port from a shaping
                        rule.
  --exclude-src-port EXCLUDE_SRC_PORT
                        exclude a specific source port from a shaping rule.

Docker:
  --docker              apply traffic control to a docker container.
  --src-container SRC_CONTAINER
                        specify source container id or name.
  --dst-container DST_CONTAINER
                        specify destination container id or name.

Documentation: https://tcconfig.rtfd.io/
Issue tracker: https://github.com/thombashi/tcconfig/issues
```




---
title: Linux系统压力测试工具
description: Linux的命令行压力测试工具在做基准测试时很有用，通过基准测试对了解一个系统所能达到的最大性能指标，这些指标可以作为后续性能比较、优化评估的参考依据。
published: true
date: 2024-11-08T02:33:30.963Z
tags: linux压力测试, linux测试
editor: markdown
dateCreated: 2024-11-08T02:33:30.963Z
---

# Linux系统压力测试工具
inux的命令行压力测试工具在做基准测试时很有用，通过基准测试对了解一个系统所能达到的最大性能指标，这些指标可以作为后续性能比较、优化评估的参考依据。

## 模拟CPU压力

可以使用 stress命令使CPU处于高负载状态。例如，通过 stress -c 4命令（-c选项用于指定CPU核心数），会让系统的4个CPU核心都处于高负载运算状态。这对于测试CPU的性能极限以及系统在CPU高负载下的响应能力很有帮助。比如，在测试服务器性能时，通过这种方式可以确定服务器在高CPU负载下是否会出现卡顿或者崩溃的情况。

模拟CPU打满：stress -c 1 -t 100
![2024-11-8_23227.png](/2024-11-8_23227.png)

`#-v 显示版本号
-v, --verbose be verbose

#-q 不显示运行信息
-q, --quiet be quiet

#-n 显示已完成的指令情况
-n, --dry-run show what would have been done

#-t --timeout N 指定运行N秒后停止

#--backoff N 等待N微妙后开始运行
-t, --timeout N timeout after N seconds
--backoff N wait factor of N microseconds before work starts

#-c 产生n个进程 每个进程都反复不停的计算随机数的平方根
-c, --cpu N spawn N workers spinning on sqrt()

#-i 产生n个进程 每个进程反复调用sync()，sync()用于将内存上的内容写到硬盘上`

## 模拟io瓶颈

fio是一个灵活且功能强大的 Linux I/O（输入 / 输出）性能测试工具。它可以对磁盘、固态硬盘（SSD）、网络存储等各种存储设备进行多种类型的 I/O 操作测试，包括但不限于顺序读写、随机读写、混合读写等，并且能够模拟不同的 I/O 负载场景。

`#随机读
fio -name=randread -direct=1 -iodepth=64 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 - runtime=1000 -group_reporting -filename=/dev/sdb

#随机写
fio -name=randwrite -direct=1 -iodepth=64 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 - runtime=1000 -group_reporting -filename=/dev/sdb

#顺序读
fio -name=read -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=4k -size=1G -numjobs=1 - runtime=1000 -group_reporting -filename=/dev/sdb

#顺序写
fio -name=write -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=4k -size=1G`

![2024-11-8_80458.png](/2024-11-8_80458.png)

`创建初始化fileio文件：
sysbench --test=fileio --file-num=16 --file-totalsize=2G prepare

接下来开始对这些文件进行测试，使用16个线程随机读进行测试结果如下：
sysbench --test=fileio --file-total-size=2G --file-testmode=rndrd --max-time=180 --maxrequests=100000000 --num-threads=16 --init-rng=on --
file-num=16 --file-extra-flags=direct --file-fsync-freq=0 -
-file-block-size=16384 run

测试结束后，记得执行cleanup,以确保测试所产生的文件都已删除：
sysbench --test=fileio --file-num=16 --file-totalsize=2G cleanup`

## 模拟大流量
iperf3是一个用于网络吞吐量测量的工具，可以测试 TCP、UDP 或 SCTP 的吞吐量。
![2024-11-8_20634.png](/2024-11-8_20634.png)

`客户端
#向目的地址10.20.81.33、5002号TCP端口，发一条TCP流，打印间隔为2s，发包时间为1000s
iperf3 -c 10.20.81.33 -p 5002 -i 2 -t 1000

服务端 
iperf3 -s -p 5002 -i 2`

## 模拟端口禁用
`#查看禁用列表
iptables -L -n --line-number

#禁用出口端口
iptables -A OUTPUT -p tcp --sport 18004 -j DROP

#禁用入口端口
iptables -A INPUT -p tcp --dport 18004 -j DROP

#删除入口端口编号
iptables -D INPUT 1

#删除出口端口编号
iptables -D OUTPUT 1`

Stress工具还提供了对内存，磁盘I/O做压力测试的命令。

Stress-ng是stress的增强版。

Sysbench主要用于数据库服务器（如MySQL）的性能测试，但也可以用于测试系统的CPU、内存和磁盘I/O性能。
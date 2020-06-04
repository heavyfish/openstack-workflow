---
description: Crash分析的vmcore来源于kdump，此处将简要介绍kdump和crash
---

# Crash调试

## 1、简介

### 1.1、kdump

Kdump是一种内核崩溃转储机制，可让您保存系统内存的内容以供以后分析。 它依赖于kexec，可用于从另一个内核的上下文引导Linux内核，绕过BIOS并保留第一个内核的内存内容，否则这些内容将丢失。

万一系统崩溃，kdump将使用kexec引导到第二个内核（即捕获内核）。 第二个内核位于第一​​个内核无法访问的系统内存的保留部分中。 然后，第二个内核捕获崩溃的内核内存的内容（崩溃转储）并保存。

### 1.2、crash

Crash 是由 Dave Anderson 开发和维护的一个内存转储分析工具，目前它的最新版本是 5.0.0。 在没有统一标准的内存转储文件的格式的情况下，Crash 工具支持众多的内存转储文件格式，包括：

* Live linux 系统
* kdump 产生的正常的和压缩的内存转储文件
* 由 makedumpfile 命令生成的压缩的内存转储文件
* 由 Netdump 生成的内存转储文件
* 由 Diskdump 生成的内存转储文件
* 由 Kdump 生成的 Xen 的内存转储文件
* IBM 的 390/390x 的内存转储文件
* LKCD 生成的内存转储文件
* Mcore 生成的内存转储文件

## 2、kdump

### 2.1、简介

kdump从RHEL5以来，已被默认安装，其对应包为`kexec-tools`。

为了使kdump顺利运行，`生产内核`会保留一定的内存给`捕获内核`使用，查看`/etc/grub2.cfg`可见：

![](../.gitbook/assets/image%20%2811%29.png)

`crashkernel=auto`即自动设置保留的内存，当然也可自己指令

### 2.2、配置

`kdump`的默认配置文件在`/etc/kdump.conf`下。其默认有两行生效配置

```text
# core dump 被存储的位置
path /var/crash

# 使用 makedumpfile 来压缩收集到的数据，
# -l 表示 启用 dump file 压缩
# -d 31 表示要忽略的内存页面数据，此处表示只导出内核态的数据
core_collector makedumpfile -l --message-level 1 -d 31
```

因为一般生成的 `vmcore` 文件仅仅是内核崩溃时的一部分信息, 如果全部导出对磁盘和时间都是很大的消耗, 默认情况下, dump 的级别为 31, 仅导出内核态的数据。下表为忽略的内存页面对应的值

**Supported Filtering Levels**

| Option | Description |
| :--- | :--- |
| `1` | Zero pages |
| `2` | Cache pages |
| `4` | Cache private |
| `8` | User pages |
| `16` | Free pages |

### 2.3、kdump原理

kexec是kdump机制的关键，包含两部分：

内核空间的系统调用kexec\_load。负责在生产内核启动时将捕获内核加载到指定地址。

用户空间的工具kexec-tools。将捕获内核的地址传递给生产内核，从而在系统崩溃的时候找到捕获内核的地址并运行。

kdump是一种基于kexec的内核崩溃转储机制。当系统崩溃时，kdump使用kexec启动到第二个内核。第二个内核通常

叫做捕获内核，以很小内存启动以捕获转储镜像。第一个内核保留了内存的一部分给第二个内核启动使用。

由于kdump利用kexec启动捕获内核，绕过了BIOS，所以第一个内核的内存得以保留。这是内存崩溃转储的本质。

捕获内核启动后，会像一般内核一样，去运行为它创建的ramdisk上的init程序。而各种转储机制都可以事先在init中实现。

为了在生产内核崩溃时能顺利启动捕获内核，捕获内核以及它的ramdisk是事先放到生产内核的内存中的。

生产内核的内存是通过/proc/vmcore这个文件交给捕获内核的。为了生成它，用户工具在生产内核中分析出内存的使用和

分布等情况，然后把这些信息综合起来生成一个ELF头文件保存起来。捕获内核被引导时会被同时传递这个ELF文件头的

地址，通过分析它，捕获内核就可以生成出/proc/vmcore。有了/proc/vmcore这个文件，捕获内核的ramdisk中的脚本就

可以通过通常的文件读写和网络来实现各种策略了。

## 3、Crash

### 3.1、yum源

```text
[debug]
name=CentOS-$releasever - DebugInfo
baseurl=http://debuginfo.centos.org/$releasever/$basearch/
gpgcheck=0
enabled=1
protect=1
priority=1
```

### 3.2、安装

首先确认故障机器的内核版本

```text
[root@sxr ~]# uname -r
3.10.0-957.21.3.el7.x86_64
```

然后安装对应的debuginfo包

```text
yum install kernel-debuginfo-3.10.0-957.21.3.el7.x86_64

#注：kernel-debuginfo-common会被作为依赖而顺便安装

yum install crash
```

确保运行`crash`机器的内核版本和故障机器的内核版本至少在同一个大版本下。

### 3.3、分析

进入`/usr/lib/debug/lib/modules/3.10.0-693.5.2.el7.x86_64/`下，把`vmcore`也拷过去，然后执行命令 `crash vmlinux vmcore` 开始分析：

进入过后，最先出现的是系统基本信息

![](../.gitbook/assets/image%20%283%29.png)

`bt`可看到堆栈, \#是调用到的堆栈

![](../.gitbook/assets/image%20%2826%29.png)

`bt -slf`显示函数偏移，函数所在的文件和每一帧的具体内容，从而对照源码和汇编代码，查看函数入参和局部变量

![](../.gitbook/assets/image%20%2842%29.png)

`dis`命令进行反汇编，查看对应地址的代码逻辑

![](../.gitbook/assets/image%20%282%29.png)

`files`查看进程打开的文件

![](../.gitbook/assets/image%20%2856%29.png)

## 4、参考文档

[对Tainted的介绍](https://www.kernel.org/doc/html/latest/admin-guide/tainted-kernels.html)

![](../.gitbook/assets/image%20%2829%29.png)

[crash\(8\)](https://linux.die.net/man/8/crash)

[crash部分命令介绍](https://www.jianshu.com/p/ad03152a0a53)

[redhat介绍](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/kernel_administration_guide/kernel_crash_dump_guide#chap-introduction-to-kdump)

[crash调试指南](https://github.com/xuliker/kde/issues/65)




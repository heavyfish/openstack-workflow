---
description: 本文从红帽文档翻译而来
---

# Red Hat 7-管理员指南

## 1、基础系统配置

本部分介绍基本的安装后任务和基本的系统管理任务，例如键盘配置，日期和时间配置，管理用户和组以及获得特权。

### 1.1、开始配置

这里有价值的只有[Kickstart Commands and Options ](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax#sect-kickstart-commands)

### 1.2、配置Data/Time

出于多种原因，保持精确的时间很重要。 在Red Hat Enterprise Linux 7中，`NTP`协议可确保时间正常，该协议由在用户空间中运行的守护程序实现。 用户空间守护程序更新内核中运行的系统时钟。 系统时钟可以通过使用各种时钟源来维护时间。

Red Hat Hat Linux 7使用以下守护程序来实现`NTP`：

* `chronyd`

默认情况下使用 `chronyd` 。 可以从`chrony`包中获得。 有关通过`chronyd`配置和使用`NTP`的更多信息, see [Chapter 18, _Configuring NTP Using the chrony Suite_](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-Configuring_NTP_Using_the_chrony_Suite).

* `ntpd`

`ntpd`守护程序可从`ntp`软件包中获得。 有关使用`ntpd`配置和使用`NTP`的更多信息, see [Chapter 19, _Configuring NTP Using ntpd_](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-Configuring_NTP_Using_ntpd).

如果要使用`ntpd`而不是默认`chronyd`，则需要禁用`chronyd`

要显示当前日期和时间，请使用以下命令之一：

```text
$ date
$ timedatectl
```

![](../.gitbook/assets/image%20%2860%29.png)

请注意，`timedatectl`命令提供了更详细的输出，包括通用时间，当前使用的时区，网络时间协议（NTP）配置的状态以及一些其他信息。

### 1.3、本地化（system locales）配置

系统范围的本地化设置存储在`/etc/locale.conf`文件中，该文件由`systemd`守护程序在早期引导时读取。 `/etc/locale.conf`中配置的本地设置由每个服务或用户继承，除非单个程序或单个用户覆盖它们。

处理系统本地化的基本任务：

* 列出可用的系统本地化

```text
localectl list-locales
```

* 显示当前的系统本地化设置

```text
localectl status
```

![](../.gitbook/assets/image%20%285%29.png)

* 设置或更改默认系统本地化设置

```text
localectl set-locale LANG=locale
```

### 1.4、键盘布局配置

```text
localectl list-keymaps
localectl status
localectl set-keymap
```

### 1.5、安装软件

```text
#搜索与特定字符串匹配的软件包
yum search string

#安装
yum install package_name

#更新所有的package
yum update

#更新某个package
yum update package_name

#列出所有已安装和可用的软件包
yum list all

#列出所有已安装的软件包
yum list installed
```

### 1.6、基础用户管理

为特定系统的用户创建普通帐户。 可以在常规系统管理期间添加，删除和修改此类帐户。

系统帐户代表系统上的特定应用程序标识符。 通常仅在软件安装时添加或操作此类帐户，以后不会对其进行修改。

{% hint style="danger" %}
假定系统帐户在系统上本地可用。 如果这些帐户被配置为可远程登录的（例如在LDAP配置实例中），则可能会发生系统故障和服务启动失败
{% endhint %}

对于系统帐户，保留低于1000的用户ID。 对于普通帐户，您可以使用从1000开始的ID。但是，建议的做法是分配从5000开始的ID。有关更多信息，参见[Section 4.1, “Introduction to Users and Groups”](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-Managing_Users_and_Groups#s1-users-groups-introduction)。 分配ID的准则可以在`/etc/login.defs`文件中找到。

```text
# Min/max values for automatic uid selection in useradd
#
UID_MIN                  1000
UID_MAX                 60000
# System accounts
SYS_UID_MIN               201
SYS_UID_MAX               999
```

```text
#显示user/group的ID
id

#添加用户到一个group
usermod -a -G group_name user_name
```

### 1.7、KDUMP机制

如果发生系统崩溃，则可以使用称为**kdump**的内核崩溃转储机制，该机制使您可以保存系统内存的内容以供以后分析。 **kdump**机制依赖于kexec系统调用，该系统调用可用于从另一个内核的上下文引导Linux内核，绕过BIOS并保留第一个内核的内存内容，否则这些内容将丢失。

发生内核崩溃时，**kdump**使用kexec引导到第二个内核（捕获内核），该内核位于第一​​个内核无法访问的系统内存的保留部分中。 第二个内核捕获崩溃的内核内存（崩溃转储）的内容并保存。

```text
#检查kdump是否暗转
rpm -q kexec-tools
```

### 1.8、REAR

当软件或硬件故障中断操作系统时，您需要一种机制来救援系统。 保存系统备份也很有用。 Red Hat建议使用 Relax-and-Recover（ReaR）工具来满足这两个需求。

ReaR是一个灾难恢复和系统迁移实用程序，可让您创建完整的救援系统。 默认情况下，此救援系统仅还原存储布局和引导加载程序，而不还原实际的用户和系统文件。

此外，某些备份软件使您能够集成ReaR以进行灾难恢复。

ReaR可以执行以下任务：

* 在新硬件上启动救援系统 
* 复制原始存储布局 
* 恢复用户和系统文件

其余知识参见链接：

[REAR-redhat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-creating_syst_backup)

[centos7使用ReaR](https://www.cnblogs.com/augusite/p/11534642.html)

### 1.9、log files介绍

Red Hat Hat Linux 7中的日志记录系统基于内置的**syslog**协议。 特定程序使用此系统记录事件并将其组织到日志文件中，这在审核操作系统和解决各种问题时很有用。

系统日志消息由两个服务处理：

* `systemd-journald`守护进程，它会从 内核，启动过程的早期阶段，守护程序启动和运行时的标准输出和标准错误输出以及syslog 中收集消息，并将消息转发到rsyslog服务以进行进一步处理。
* `rsyslog`服务-按类型和优先级对syslog消息进行排序，然后将其写入`/ var / log`目录中的文件中，日志将永久存储在该文件中

syslog消息根据它们包含的消息和日志类型存储在`/var/log`目录下的各个子目录中：

* `var/log/messages` - 除了下面提到的那些的所有syslog messages
* `var/log/secure` - 与安全和身份验证相关的消息和错误
* `var/log/maillog` - 邮件服务器相关的消息和错误
* `var/log/cron` - 与定期执行的任务有关的日志文件
* `var/log/boot.log` - 与系统启动有关的日志文件

## 2、系统本地化和键盘布局

系统本地化（system locale）设置 指定系统服务和用户界面的语言设置。 键盘布局设置 控制在文本控制台和图形用户界面上使用的布局。

可以通过修改`/etc/locale.conf`配置文件或使用**localectl**实用程序来进行这些设置。

### 2.1、系统本地化配置

系统范围的本地化设置存储在`/etc/locale.conf`文件中，该文件由`systemd`守护程序在早期引导时读取。 `/etc/locale.conf`中配置的本地设置由每个服务或用户继承，除非单个程序或单个用户覆盖它们。

`/etc/locale.conf`的基本文件格式是用换行符分隔的变量列表。 例如，在`/etc/locale.conf`中带有英语消息的德语本地化如下所示：

```text
LANG=de_DE.UTF-8
LC_MESSAGES=C
```

在此，LC\_MESSAGES选项确定用于写入标准错误输出的诊断消息的语言设置。 要在`/etc/locale.conf`中进一步指定本地化设置，可以使用其他几个选项，请注意，代表所有可能选项的LC\_ALL选项不应在`/etc/locale.conf`中进行配置。

| Option | Description |
| :--- | :--- |
| LANG | 提供 system locale 的默认值 |
| LC\_COLLATE | 更改比较本地字母中的字符串的功能的行为 |
| LC\_CTYPE | 更改字符处理和分类功能以及多字节字符功能的行为. |
| LC\_NUMERIC | 描述数字的通常打印方式，并带有小数点和小数逗号等详细信息。 |
| LC\_TIME | 更改当前时间的显示（24小时制与12小时制） |
| LC\_MESSAGES | 确定用于写入标准错误输出的诊断消息的语言环境. |

```text
localectl status
localectl list-locales
localectl set-locale LANG=locale
```

使用Kickstart进行安装时使system locale 设置永久不变（[链接](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-keyboard_configuration)）

### 2.2、改变键盘布局

[链接](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/s1-changing_the_keyboard_layout)

```text
localectl list-keymaps

#设置默认键盘布局
localectl set-keymap map
```

## 3、配置时间

现代操作系统区分以下两种时钟类型：

* _real-time clock_ \(RTC\), 通常称为硬件时钟（通常是系统板上的集成电路），它完全独立于操作系统的当前状态，并且即使在计算机关闭时也可以运行
* _system clock_, 也称为软件时钟，由内核维护，其初始值基于 `real-time clock`一旦启动系统并初始化了`system clock`，系统时钟便完全独立于`real-time clock`

系统时间始终维护为世界标准时间（UTC），并在应用程序中根据需要将其转换为本地时间。 本地时间是考虑了夏令时（DST）的当前时区中的实际时间。 实时时钟（real-time clock）可以使用UTC或本地时间。 建议使用UTC。

Red Hat Enterprise Linux 7提供了三个命令行工具，可用于配置和显示有关系统日期和时间的信息：

* The `timedatectl` utility, which is new in Red Hat Enterprise Linux 7 and is part of `systemd`.
* The traditional `date` command.
* The `hwclock` utility for accessing the hardware clock.

### 3.1、timedatectl

**timedatectl**作为`systemd`系统和服务管理器的一部分分发，使您可以查看和更改系统时钟的配置。 您可以使用此工具更改当前日期和时间，设置时区或启用系统时钟与远程服务器的自动同步。

![](../.gitbook/assets/image%20%2836%29.png)

{% hint style="danger" %}
`chrony`或`ntpd`的状态更改不会立即被`timedatectl`注意到。 如果更改了这些工具的配置或状态，请输入以下命令：

```text
systemctl restart systemd-timedated.service
```
{% endhint %}

改变当前时间

```text
# HH:时  MM:分 SS:秒
timedatectl set-time HH:MM:SS
timedatectl set-time 23:26:00

timedatectl set-time YYYY-MM-DD
timedatectl set-time 2017-06-02 23:26:00 

#改变时区
timedatectl set-timezone time_zone
```

此命令同时更新系统时间和硬件时钟。 结果类似于使用`date --set`和`hwclock --systohc`命令。

如果启用了`NTP`服务，该命令将失败。

默认情况下，系统配置为使用UTC。 要将系统配置为将时钟维持在local time，`timedatectl`命令带上`set-local-rtc`选项：

```text
timedatectl set-local-rtc boolean
```

要将系统配置为将时钟维持在本地时间，请将boolean替换为`yes`（或者是`y`，`true`，`t`或`1`）。 要将系统配置为使用UTC，请用`no`（或`n`，`false`，`f`或`0`）。 默认选项是“`no`”。

timedatectl命令还允许您使用`NTP`协议启用系统时钟与一组远程服务器的自动同步。 启用`NTP`会启用`chronyd`或`ntpd`服务，这取决于已安装的服务。

```text
timedatectl set-ntp yes/no
```

### 3.2、date

date实用程序可在所有Linux系统上使用，并允许您显示和配置当前日期和时间。 它通常用于脚本中，以自定义格式显示有关系统时钟的详细信息。

```text
[root@sxr ~]# date
Fri Mar 13 17:37:01 CST 2020

#显示UTC 时间
date --utc

date +"format"

date --set HH:MM:SS <--utc>
date --set "2017-06-02 23:26:00"
```

| Control Sequence | Description |
| :--- | :--- |
| `%H` | The hour in the _HH_ format \(for example, `17`\). |
| `%M` | The minute in the _MM_ format \(for example, `30`\). |
| `%S` | The second in the _SS_ format \(for example, `24`\). |
| `%d` | The day of the month in the _DD_ format \(for example, `16`\). |
| `%m` | The month in the _MM_ format \(for example, `09`\). |
| `%Y` | The year in the _YYYY_ format \(for example, `2016`\). |
| `%Z` | 时区缩写 \(for example, `CEST`\). |
| `%F` | The full date in the _YYYY-MM-DD_ format \(for example, `2016-09-16`\). This option is equal to `%Y-%m-%d`. |
| `%T` | The full time in the _HH:MM:SS_ format \(for example, 17:30:24\). This option is equal to `%H:%M:%S` |

### 3.2、hwclock

hwclock是用于访问硬件时钟（也称为Real Time（RTC））的实用程序。 硬件时钟与您使用的操作系统无关，即使机器关闭也可以工作。 该实用程序用于显示硬件时钟的时间。 `hwclock`还包含用于补偿硬件时钟的系统漂移（systematic drift）的功能。

硬件时钟存储以下值：年，月，日，时，分，秒。 它无法存储时间标准，本地时间或世界标准时间（UTC），也无法设置夏令时（DST）。

`hwclock`实用程序将其设置保存在`/etc/adjtime`文件中，该文件是通过您进行的第一次更改创建的，例如，当您手动设置时间或将硬件时钟与系统时间同步时。

{% hint style="info" %}
在Red Hat Enterprise Linux 6中，每次系统关闭或重新启动时都会自动运行hwclock命令，但是在Red Hat Linux 7中则没有。当系统时钟通过网络时间协议（NTP）或精确时间协议（ PTP）同步时，内核每11分钟自动将硬件时钟与系统时钟同步一次。
{% endhint %}

请注意，在`hwclock`命令中使用`--utc`或-`-localtime`选项并不表示您以UTC或本地时间显示硬件时钟时间。 这些选项用于设置硬件时钟以什么方式维持时间

```text
hwclock --set --date "dd mmm yyyy HH:MM"
```

您还可以通过分别添加`--utc`或`--localtime`选项来设置硬件时钟，以使时间维持UTC或本地时间。 在这种情况下，UTC或LOCAL记录在`/etc/adjtime`文件中。

```text
#将硬件时钟设置为当前系统时间
hwclock --systohc

#将系统时间设置为当前硬件时钟
hwclock --hctosys
```

## 4、用户和组

忽略（[链接](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-managing_users_and_groups)）

## 5、ACL

文件和目录具有文件所有者，与文件关联的组以及系统的所有其他用户的权限集。 但是，这些权限集有局限性。 例如，不能为不同的用户配置不同的权限。 因此，实现了访问控制列表（ACL）。（[链接](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-access_control_lists)）

## 6、普通用户获得特权

系统管理员以及某些情况下的用户，需要通过管理员权限执行某些任务。 以root用户身份访问系统具有潜在的危险，并且可能导致对系统和数据的广泛破坏。 本章介绍使用`setuid`程序（例如`su`和`sudo`）获得管理特权的方法。 这些程序允许特定用户执行通常仅对`root`用户可用的任务，同时保持更高级别的控制和系统安全性。

### 6.1、su

当用户执行`su`命令时，系统会提示他们输入root密码，并在身份验证后向其提供root shell提示。

使用su命令登录后，该用户即为root用户，并且具有对该系统的绝对管理访问权限。 请注意，此访问仍受SELinux施加的限制。 此外，一旦用户成为root用户，他们就可以使用su命令更改为系统上的任何其他用户，而无需提示输入密码。

最简单的方法之一是将用户添加到名为wheel的特殊管理组中。 为此，请以超级用户身份键入以下命令：

```text
~]# usermod -a -G wheel username
```

将所需用户添加到`Wheel`组后，建议仅允许这些特定用户使用`su`命令。 为此，请编辑可插拔身份验证模块（PAM）的`/etc/pam.d/su`配置文件。 在文本编辑器中打开此文件，并通过删除＃字符取消注释以下行：

```text
#auth           required        pam_wheel.so use_uid
```

此更改意味着只有管理组`wheel`的成员才能使用su命令切换到另一个用户。

{% hint style="info" %}
默认情况下，root用户在wheel组中
{% endhint %}

### 6.2、sudo

`sudo`命令提供了另一种向用户提供管理访问权限的方法。 当受信任的用户在使用sudo的管理命令之前时，将提示他们输入自己的密码。 然后，在对它们进行身份验证并假定命令被允许后，将像他们是root用户一样执行管理命令。

```text
sudo command
```

`sudo`命令具有高度的灵活性。 例如，仅允许`/etc/sudoers`配置文件中列出的用户使用sudo命令，并且该命令在用户的shell中执行，而不是在root shell中执行。 这意味着root shell 可以完全禁用。

使用`sudo`命令的每次成功身份验证都会记录到文件`/var/log/messages`中，并且执行的命令将与使用者的用户名一起记录到文件`/var/log/secure`中。 如果需要其他日志记录，请使用`pam_tty_audit`模块，通过将以下行添加到`/etc/pam.d/system-auth`文件中来为指定用户启用TTY审计：

```text
session required pam_tty_audit.so disable=pattern enable=pattern
```

其中pattern表示用逗号分隔的用户列表，并使用glob。 例如，以下配置将为root用户启用TTY审核，并为所有其他用户禁用TTY审核：

```text
session required pam_tty_audit.so disable=* enable=root
```

{% hint style="warning" %}
配置pam\_tty\_audit PAM模块以进行TTY审核时，仅记录TTY输入。 这意味着，当受审核用户登录时，pam\_tty\_audit会将用户进行的确切击键记录到/var/log/audit/audit.log文件中。
{% endhint %}

sudo命令的另一个优点是管理员可以允许不同的用户根据他们的需要访问特定的命令。

想要编辑sudo配置文件`/ etc / sudoers`的管理员应使用`visudo`命令。

要授予某人完全的管理特权，请键入`visudo`并在用户特权规范部分中添加类似于以下内容的行：

```text
juan ALL=(ALL) ALL
```

此示例说明用户`juan`可以从任何主机使用sudo并执行任何命令。

以下示例说明了配置sudo时可能的粒度：

```text
%users localhost=/usr/sbin/shutdown -h now
```

此示例说明，`users`组的任何成员可以在控制台执行`/sbin/shutdown -h now`。

 `sudoers` 的man page有关于配置文件的详细介绍。

您还可以通过使用/ etc / sudoers文件中的NOPASSWD选项来配置不需要提供任何密码的sudo用户：

```text
user_name ALL=(ALL)	NOPASSWD: ALL
```

但是，即使对于此类用户，`sudo`也会运行可插拔身份验证模块（PAM）帐户管理模块，从而可以检查PAM模块在身份验证阶段之外施加的限制。 这样可以确保PAM模块正常工作。 例如，对于`pam_time`模块，基于时间的帐户限制不会失败。

使用sudo命令时要牢记一些潜在的风险。 您可以通过使用`visudo`编辑`/etc/sudoers`配置文件来避免它们，如上所述。 将`/etc/sudoers`文件保留为默认状态可为`Wheel` Group中的每个用户提供无限制的root访问权限。

默认情况下，sudo将密码存储五分钟。 在此期间，此命令的任何后续使用都不会提示用户输入密码。 可以通过在/etc/sudoers文件中添加以下行来更改此行为：

```text
# value 是超时时间（min），设为0，则每次命令都要输入密码
Defaults    timestamp_timeout=value
```

如果帐户被盗用，攻击者可以使用sudo来以管理特权打开新的shell：

```text
sudo /bin/bash
```

以root身份或类似方式打开新的shell，使攻击者在理论上不受时间限制，从而绕过了`/etc/sudoers`文件中指定的超时期限，并且永远不需要攻击者再次输入sudo的密码，直到 新打开的会话已关闭。

## 7、安装/管理软件

yum可以很方便的安装rpm软件包，并处理依赖关系。

Yum通过为所有软件包存储库（软件包源）或单个存储库启用对GPG（Gnu Privacy Guard；也称为GnuPG）签名验证_来_提供安全的软件包管理。 启用签名验证后，yum将拒绝安装任何未使用该存储库的正确 key 进行GPG签名的软件包。 这意味着您可以相信您下载并安装在系统上的RPM软件包来自受信任的来源（例如Red Hat），并且在传输过程中没有被修改。

### 7.1、升级软件

```text
# 检查可用的更新
yum check-update

#更新指定包
yum update package_name

#更新指定的软件包组
yum group update group_name

#如果软件包提供了安全更新，则更新该软件包
yum update --security

#仅将软件包更新为包含最新安全更新的版本
yum update-minimal --security
```

For example, assume that:

* the kernel-3.10.0-1 package is installed on your system;
* the kernel-3.10.0-2 package was released as a security update;
* the kernel-3.10.0-3 package was released as a bug fix update.

Then `yum update-minimal --security` updates the package to kernel-3.10.0-2, and `yum update --security` updates the package to kernel-3.10.0-3.

#### 用ISO离线更新

```text
mount -o loop iso_name mount_dir
cp mount_dir/media.repo /etc/yum.repos.d/new.repo

#修改.repo文件
baseurl=file:///mount_dir
```

### 7.2、管理包

Yum使您可以对软件包执行一整套操作，包括搜索软件包，查看有关软件包的信息，安装和删除。

#### 搜索包

```text
#搜索包
yum search package
yum search all package
```

#### 列出包

`yum` 的 `list`命令允许使用通配符，但需要注意转义。可用的通配符有`*`（匹配任何字符子集）和`?`（匹配任何单个字符）。转义方法有：

* 反斜杠（\）转义，如 `a\*`
* 双引号或单引号转义，如`“a*”`

```text
#列出所有已安装和可用软件包的信息
yum list all
yum list glob_expression(匹配式)

#列出所有已安装的
yum list installed
yum list glob_expression

#列出所有可用的
yum list available glob_expression…
```

#### 列出仓库信息

```text
#列出所有启用的仓库
yum repolist

#查看仓库详细信息
yum repolist -v 
yum repoinfo 

#列出所有仓库
yum repolist all
```

#### 显示包信息

```text
yum info package_name…
```

在yum数据库中查询有关程序包的可选信息和有用信息

```text
yumdb info package_name
```

![](../.gitbook/assets/image%20%2817%29.png)

#### 安装包

```text
yum install package_name <package_name>
yum install glob_expression
```

除了软件包名称和glob表达式外，您还可以为yum安装提供文件名。 如果您知道要安装的二进制文件的名称，但不知道其软件包名称，则可以给`yum install`路径名，然后Yum搜索其软件包列表，找到提供/ usr / sbin / named的软件包（如果有），并提示您是否要安装它

```text
yum install /usr/sbin/named
```

在`install-n`中，yum将名称解释为软件包的确切名称。 `install-na`命令告诉yum，后面的参数包含 软件包名称和体系结构，以点字符分隔。 在`install-nevra`中，yum将期望使用`name-epoch:version-release.architecture`形式的参数。 同样，在搜索要删除的软件包时，可以使用`yum remove-n`，`yum remove-na`和`yum remove-nevra`。

```text
yum install-n name
yum install-na name.architecture
yum install-nevra name-epoch:version-release.architecture
```

{% hint style="info" %}
如果您知道要安装 包含命名二进制文件的软件包，但不知道文件安装在bin/或sbin/目录中，请使用带有全局表达式的yum provides命令，如：

```text
~]# yum provides "*bin/named"
```
{% endhint %}

要从系统上的本地目录安装以前下载的软件包：

```text
#用要安装的软件包的路径替换path
yum localinstall path
```

#### 下载包

交互式安转时：

```text
...
Total size: 1.2 M
Is this ok [y/d/N]:    
...
```

使用`d`选项，yum将下载软件包，而无需立即安装它们。 您可以稍后使用`yum localinstall`命令脱机安装这些软件包，也可以与其他设备共享它们。 下载的软件包默认保存在缓存目录的子目录之一中，即`/var/cache/yum/$basearch/$releasever/packages/`。 下载在后台模式下进行，因此您可以并行使用yum进行其他操作。

#### 移除包

```text
yum remove package_name…
```

Similar to `install`, `remove` can take these arguments:

* package names
* glob expressions
* file lists
* package provides

{% hint style="danger" %}
yum在删除软件包时，必然会删除其依赖的软件包。所有移除软件包最好用rpm命令
{% endhint %}

### 7.3、管理package group

程序包组是用于通用目的的程序包的集合，例如系统工具或声音和视频。 安装软件包组可以拉出一组相关的软件包，从而可以节省大量时间。 `yum groups`命令是一个顶层命令，它涵盖了所有对yum软件包组起作用的操作。（[链接](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-working_with_package_groups)\)

### 7.4、Transaction history

`yum history`命令使用户可以查看有关yum事务的时间轴，发生的日期和时间，受影响的软件包数量，这些事务成功还是中止以及RPM数据库是否在事务之间被更改。 此外，此命令可用于撤消或重做某些事务。 所有历史记录数据都存储在`/var/lib/yum/history/`目录中的历史记录数据库中。

```text
yum history list
yum history list all
yum history list start_id..end_id
yum history list glob_expression…
```

![](../.gitbook/assets/image%20%2839%29.png)

**Table 9.1. Possible values of the Action\(s\) field**

| Action | Abbreviation | Description |
| :--- | :--- | :--- |
| `Downgrade` | `D` | 至少一个软件包已降级为较旧的版本. |
| `Erase` | `E` | 至少一个软件包被移除. |
| `Install` | `I` | At least one new package has been installed. |
| `Obsoleting` | `O` | 至少有一个包装已被标记为已过时。. |
| `Reinstall` | `R` | 至少一个软件包已重新安装。. |
| `Update` | `U` | At least one package has been updated to a newer version. |

**Table 9.2. Possible values of the Altered field**

| Symbol | Description |
| :--- | :--- |
| `<` | 在事务完成之前，在yum之外更改了`rpmdb`数据库. |
| `>` | 在事务完成后，在yum之外更改了rpmdb数据库. |
| `*` | The transaction failed to finish. |
| `#` | The transaction finished successfully, but yum returned a non-zero exit code. |
| `E` | 事务成功完成，但是显示了错误或警告 |
| `P` | 事务成功完成，但是`rpmdb`数据库中已经存在问题. |
| `s` | 事务成功完成，但是使用了`--skip-broken`命令行选项，并且跳过了某些软件包. |

要将任何已安装软件包的`rpmdb`或`yumdb`数据库内容与当前使用的`rpmdb`或`yumdb`数据库同步，请键入以下内容：

```text
yum history sync
```

要显示有关当前使用的历史数据库的一些总体统计信息，请使用以下命令：

```text
yum history stats
```

显示所有过去事务的摘要：

```text
yum history summary
yum history summary start_id..end_id
yum history summary glob_expression…
```

![](../.gitbook/assets/image%20%289%29.png)

从软件包的角度列出事务：

```text
yum history package-list glob_expression…
```

#### 检查事务

```text
yum history summary id
yum history info id…
yum history info start_id..end_id
yum history addon-info id
yum history addon-info last
```

![](../.gitbook/assets/image%20%2828%29.png)

* `config-main` — global yum options that were in use during the transaction.
* `config-repos` — 各个yum存储库的选项. 
* `saved_tx` — `yum load-transaction`命令可以使用的数据，以便在另一台机器上重复该事务.

### 7.5、还原事务

```text
#还原
yum history undo id

#重复
yum history redo id
```

请注意，`yum history undo`和`yum history redo`命令都仅还原或重复在事务期间执行的步骤。 如果该事务安装了新软件包，则yum history undo命令将其卸载；如果该事务卸载了软件包，则该命令将再次安装它。 如果这些较旧的软件包仍然可用，则此命令还将尝试将所有更新的软件包降级到其先前版本。

当管理多个相同的系统时，yum还使您能够在其中一个系统上执行事务，将事务详细信息存储在文件中，经过一段时间的测试后，还可以在其余系统上重复相同的事务。 要将交易详细信息存储到文件中，键入以下内容：

```text
yum -q history addon-info id saved_tx > file_name
yum load-transaction file_name
```

#### 启用新的事务历史

Yum将事务历史存储在单个SQLite数据库文件中。 要启动新的事务历史：

```text
yum history new
```

这将在`/var/lib/yum/history/`目录中创建一个新的空数据库文件。 旧的事务历史记录将保留，但只要目录中存在较新的数据库文件，就将无法访问。

### 7.5、配置yum

yum和相关实用程序的配置信息位于`/etc/yum.conf`。 该文件包含一个必填的`[main]`节，使您可以设置具有全局作用的yum选项，还可以包含一个或多个`[repository]`，使您可以设置特定于存储库的选项。 但是，建议在`/etc/yum.repos.d/`目录中新的或现有的`.repo`文件中定义各个存储库。 您在`/etc/yum.conf`文件的各个`[repository]`部分中定义的值将覆盖\[main\]部分中设置的值。

#### 设置\[main\]选项

`/etc/yum.conf`配置文件仅包含一个`[main]`部分，该部分中的某些键值对影响yum的运行方式，而其他一些影响yum对待存储库的方式。 您可以在/etc/yum.conf中\[main\]部分下添加许多其他选项。

#### `assumeyes`=_value_

The `assumeyes` yum是否提示您确认关键操作:

`0` \(_default_\) — yum提示确认其执行的关键操作.

`1` — 不要提示您确认关键的yum操作。 如果设置了 `assumeyes=1`  ，则yum的行为与命令行选项 `-y` and `--assumeyes`相同.

#### `cachedir`=_directory_

使用此选项设置yum存储其缓存和数据库文件的目录。 用目录的绝对路径替换目录。 默认情况下，yum的缓存目录为`/var/cache/yum/$basearch/$releasever`

#### `debuglevel`=_value_

此选项指定yum产生的调试输出的详细信息。 此处，value是1到10之间的整数。设置较高的调试级别值会使yum显示更详细的调试输出。 默认值为debuglevel = 2，而debuglevel = 0禁用调试输出。

#### `exactarch`=_value_

使用此选项，可以将yum设置为在更新已安装的软件包时考虑确切的体系结构

`0`-更新软件包时不考虑确切的体系结构。

`1`（default）—更新软件包时，请考虑确切的体系结构。 使用此设置，yum不会用32位体系结构的软件包来更新64位体系结构的系统上的软件包。

#### `exclude`=_package\_name_ \[_more\_package\_names_\]

`exclude`选项使您可以在安装或系统更新期间按关键字排除软件包。 列出多个要排除的软件包可以通过引用以空格分隔的软件包列表来实现。 允许使用通配符（例如\*和？）的Shell glob表达式。

#### `gpgcheck`=_value_

使用`gpgcheck`选项可以指定yum是否应该对软件包执行GPG签名检查。

`0`-对所有存储库中的软件包禁用GPG签名检查，包括本地软件包安装。

`1`（默认）-启用所有存储库中所有软件包的GPG签名检查，包括本地软件包安装。 启用gpgcheck后，将检查所有软件包的签名。

如果在`/etc/yum.conf`文件的\[main\]部分中设置了此选项，则它将为所有存储库设置GPG检查规则。 但是，您也可以在`.repo`文件中为单个存储库设置`gpgcheck = value`； 也就是说，您可以在一个存储库上启用GPG检查，而在另一个存储库上禁用它。

#### `group_command`=_value_

Use the `group_command` option to specify how the `yum group install`, `yum group upgrade`, and `yum group remove` commands handle a package group. Replace _value_ with on of:

`simple` — Install all members of a package group. Upgrade only previously installed packages, but do not install packages that have been added to the group in the meantime.

`compat` — Similar to `simple` but `yum upgrade` also installs packages that were added to the group since the previous upgrade.

`objects` — \(_default_.\) With this option, yum keeps track of the previously installed groups and distinguishes between packages installed as a part of the group and packages installed separately

#### `group_package_types`=_package\_type_ \[_more\_package\_types_\]

在这里，您可以指定在调用yum group install命令时安装的软件包类型（_optional_, _default_ or _mandatory_）。 默认情况下选择_default_和_mandatory_软件包类型。

#### `history_record`=_value_

使用此选项，您可以将yum设置为记录_事务_历史记录

`0`-yum不记录。

`1`（默认）— yum应该记录事务历史。 该操作占用一定数量的磁盘空间，并在事务中花费一些额外的时间，但是它提供了许多有关过去操作的信息，这些信息可以用yum history命令显示。

{% hint style="info" %}
Yum使用历史记录来检测在yum之外对rpmdb数据库所做的修改。 在这种情况下，yum将显示警告并自动搜索由更改rpmdb引起的可能问题。 关闭history\_record时，yum无法检测到这些更改，因此不会执行自动检查。
{% endhint %}

#### `installonlypkgs`=_space_ _separated_ _list_ _of_ _packages_

您可以在此处提供以空格分隔的软件包列表，yum可以安装这些软件包，但永远不会更新

#### `installonly_limit`=_value_

此选项设置可以同时安装`installonlypkgs`指令中列出的软件包数量。 用整数替换值，该整数代表可以在`installonlypkgs`中列出的任何单个软件包同时安装的最大版本数_。_

`installonlypkgs`指令的缺省值代表拥有几个不同的内核程序包，因此请注意，更改`installonly_limit`的值还会影响任何单个内核程序包的最大已安装版本数。 /etc/yum.conf中列出的缺省值为`installonly_limit` = 3，可能的最小值为`installonly_limit` = 2。

您不能设置`installonly_limit` = 1，因为这会使yum删除正在运行的内核，这是被禁止的。 如果使用`installonly_limit` = 1，则yum失败。

#### `keepcache`=_value_

`keepcache`选项确定yum在成功安装后是否保留headers和程序包的高速缓存。

`0`（默认）—成功安装后，不保留headers和程序包的高速缓存。

`1`成功安装后保留缓存。

#### `logfile`=_file\_name_

指定日志输出位置

#### `max_connenctions`=_number_

此处的值表示最大同时连接数，默认值为5。

#### `multilib_policy`=_value_

如果软件包可以使用多个体系结构版本，则multilib\_policy选项将设置安装行为。

`best` — install the best-choice architecture for this system. For example, setting `multilib_policy=best` on an AMD64 system causes yum to install the 64-bit versions of all packages.

`all` — always install every possible architecture for every package. For example, with `multilib_policy` set to `all` on an AMD64 system, yum would install both the i686 and AMD64 versions of a package, if both were available.

#### `obsoletes`=_value_

`obsoletes`选项启用更新过程中的废弃处理逻辑。当一个软件包在其spec文件中声明已废弃另一个软件包时，安装前一个软件包时，后一个软件包将由前一个软件包替换。

`0` — Disable yum's obsoletes processing logic when performing updates.

`1` \(_default_\) — Enable yum's obsoletes processing logic when performing updates.

#### `plugins`=_value_

这是启用或禁用yum插件的全局开关：

`0` — Disable all yum plug-ins globally.  
`1`（默认）—全局启用所有yum插件。 使用plugins = 1时，您仍然可以通过在该插件的配置文件中设置`enabled = 0`来禁用特定的yum插件

#### `reposdir`=_directory_

.repo文件所在目录的绝对路径

#### `retries`=_value_

此选项设置yum在返回错误之前应尝试检索文件的次数。值为0或更大的整数。将该值设置为0会使yum永远重试。预设值为10






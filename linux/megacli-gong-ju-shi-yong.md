---
description: MegaRaid阵列卡管理工具。它可以查看当前RAID卡的所有信息，包括RAID卡型号、类型、磁盘状态、电池状态等等。
---

# Megacli工具使用

## 1、下载与安装

### 1.1、戴尔官网下载

{% embed url="http://www.dell.com/support/article/cn/zh/cndhs1/SLN29223" %}

### 1.2、Megacli官网下载

{% embed url="https://www.broadcom.com/support/download-search" caption="" %}

![&#x8FDB;&#x5165;&#x5B98;&#x7F51;&#x641C;&#x7D22;](../.gitbook/assets/image%20%281%29.png)

![&#x70B9;&#x51FB;&#x4E0B;&#x8F7D;](../.gitbook/assets/image%20%2851%29.png)

```text
#安装一下软件
rpm -ivh Linux/MegaCli-8.07.14-1.noarch.rpm  

#可以查看版本以及相关的命令使用说明
/opt/MegaRAID/MegaCli/MegaCli64 -help  

```

## 2、Megacli命令

```python
#查看raid级别
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL  

#查看raid卡信息
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aALL 

#查看硬盘信息-常用
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL 

#查看指定盘
/opt/MegaRAID/MegaCli/MegaCli64 -pdInfo -PhysDrv[32:0] -aALL 

#查看电池信息
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -aAll 

#查看raid卡日志
/opt/MegaRAID/MegaCli/MegaCli64 -FwTermLog -Dsply -aALL 

#显示适配器个数
/opt/MegaRAID/MegaCli/MegaCli64 -adpCount 

#显示适配器时间
/opt/MegaRAID/MegaCli/MegaCli64 -AdpGetTime –aALL 

#显示所有适配器信息-常用；通常注意Adapter即可
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aAll

#显示适配器数量
/opt/MegaRAID/MegaCli/MegaCli64 -adpCount

#显示所有逻辑磁盘组信息-常用
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -LALL -aAll  

#查看电池充电状态
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuStatus -aALL |grep 'Charger Status' 

#显示raid卡型号，raid设置，disk相关信息
/opt/MegaRAID/MegaCli/MegaCli64 -cfgdsply -aALL 

#闪灯
/opt/MegaRAID/MegaCli/MegaCli64 -PdLocate -start -physdrv[32:0] -a0
#关灯
/opt/MegaRAID/MegaCli/MegaCli64 -PdLocate -start -physdrv[32:0] -a0
```

### 2.1、创建RAID 0

1、新盘插进来时，一般要清除其Foregin信息 或 清除该盘在RAID卡上的缓存信息

```python
#扫描外来配置信息
/opt/MegaRAID/MegaCli/MegaCli64 -cfgforeign -scan -a0      
                                
There are 2 foreign configuration(s) on controller 0.

Exit Code: 0x00
```

```python
#清除外来配置信息
/opt/MegaRAID/MegaCli/MegaCli64 -cfgforeign -clear -a0

#在此扫描
/opt/MegaRAID/MegaCli/MegaCli64 -cfgforeign -scan -a0 
                                 
There is no foreign configuration on controller 0.

Exit Code: 0x00
```

```python
 #查看保留的缓存列表，这里的ID是02，后面应该清楚02
 ./MegaCli64 -GetPreservedCacheList -a0
Adapter #0
Virtual Drive(Target ID 02): Missing.

Exit Code: 0x00
```

```python
#清楚缓存 L2 就对应 上面的 02， a0表示适配器0
./MegaCli64 -DiscardPreservedCache -L2 -a0
Adapter #0
Virtual Drive(Target ID 02): Preserved Cache Data Cleared.

Exit Code: 0x00
```

```python
#创建RAID 0
#Adapter #0                阵列卡号,适配器编号
#Enclosure Device ID: 32   raid卡的ID号
#Slot Number: 3            物理磁盘的slot号,磁盘位置
./MegaCli64 -CfgLdAdd -r0 [32:3] -a0
Adapter 0: Created VD 2
Adapter 0: Configured the Adapter!!

Exit Code: 0x00
```

```python
#清除RIAD 0
#[id] 就是 Virtual Drive(Target ID 02)
/opt/MegaRAID/MegaCli/MegaCli64 -L[id] -aALL 
```

## 3、参考文档链接

1、扩展工具

* 目前storcli已经基本代替了megacli，由于加载驱动原因可能无法使用，然后使用megacli。
* smartmontools监控硬盘的可靠性、预测磁盘故障和执行各种类型的磁盘自检

2、文档链接

[Megacli命令介绍与操作分享](http://www.51niux.com/?id=77)

[Megacli、storcli、smartmontools介绍](http://www.51niux.com/?id=77)

[Megacli的精简介绍](https://blog.csdn.net/GX_1_11_real/article/details/83213959)（快查看这里）

[Megacli命令参考](http://www.freeoa.net/osuport/sysadmin/linux-dell-raid-tool-megacli-cmd-ref_1408.html)（较详细，格式不好）




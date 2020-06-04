---
description: NUMA是计算机中的一种计算体系架构，在OpenStack中，通过实现Nova实现的NUMA亲和，可以提高虚机的性能
---

# NUMA详解

## 1、计算平台体系结构

### 1.1、SMP对称多处理架构

**SMP（Sysmmetric Multi-Processor，对称多处理器）**，顾名思义，SMP 由多个具有对称关系的处理器组成。所谓对称，即处理器之间是水平的镜像关系，无有主从之分。SMP 的出现使一台计算机不再由单个 CPU 组成。

_**SMP 的典型特征为\*\*「多个处理器共享一个集中式存储器」\*\***_，且每个处理器访问存储器的时间片相同，使得工作负载能够均匀的分配到所有可用处理器上，极大地提高了整个系统的数据处理能力。

![](../.gitbook/assets/image%20%2815%29.png)

虽然 SMP 具有多个处理器，但由于只有一个共享的集中式存储器，所以 SMP 只能运行一个操作系统和数据库系统的副本（实例\)，依旧保持了单机特性。同时，SMP 也会要求多处理器保证共享存储器的数据一致性。如果多个处理器同时请求访问共享资源，就需要由软件或硬件实现的加锁机制来解决资源竞态的问题。由此，SMP 又称为 UMA（Uniform Memory Access，一致性存储器访问\)，所谓一致性指的是：

* **在任意时刻，多个处理器只能为存储器的每个数据保存或共享一个唯一的数值。**
* **每个处理器访问存储器所需要的时间都是一致的**

显然，这样的架构设计注定没法拥有良好的处理器数量扩展性，因为共享存储的资源竞态总是存在的，**处理器利用率最好的情况只能停留在 2 到 4 颗**。综合来看，SMP 架构广泛的适用于 PC 和移动设备领域，能显著提升并行计算性能。但 SMP 却不适合超大规模的服务器端场景，例如：云计算。

![](../.gitbook/assets/image%20%2838%29.png)

### 1.2、NUMA非统一内存访问结构

现代计算机系统中，处理器的处理速度已经超过了主存的读写速度，限制计算机计算性能的瓶颈转移到了存储器带宽之上。SMP 由于集中式共享存储器的设计限制了处理器访问存储器的频次，导致处理器可能会经常处于对数据访问的饥饿状态。

_**\*\*NUMA（Non-Uniform Memory Access，非一致性存储器访问）\*\*的设计理念是将处理器和存储器划分到不同的节点（NUMA Node），它们都拥有几乎相等的资源**_。在 NUMA 节点内部会通过自己的存储总线访问内部的本地内存，而所有 NUMA 节点都可以通过主板上的共享总线来访问其他节点的远程内存。

![](../.gitbook/assets/image%20%2822%29.png)

**很显然，处理器访问本地内存和远程内存的时耗并不一致，NUMA 非一致性存储器访问由此得名。而且因为节点划分并没有实现真正意义上的存储隔离，所以 NUMA 同样只会保存一份操作系统和数据库系统的副本。**

![](../.gitbook/assets/image%20%2818%29.png)

NUMA「**多节点**」的结构设计能够在一定程度上解决 SMP 低存储带宽的问题。假如有一个 4 NUMA 节点的系统，每一个 NUMA 节点内部具有 1GB/s 的存储带宽，外部共享总线也同样具有 1GB/s 的带宽。理想状态下，如果所有的处理器总是访问本地内存的话，那么系统就拥有了 4GB/s 的存储带宽能力，此时的每个节点可以近似的看作为一个 SMP（这种假设为了便于理解，并不完全正确）；相反，在最不理想的情况下，如果所有处理器总是访问远程内存的话，那么系统就只能有 1GB/s 的存储带宽能力。

除此之外，使用外部共享总线时可能会触发 NUMA 节点间的 Cache 同步异常，这会严重影响内存密集型工作负载的性能。当 I/O 性能至关重要时，共享总线上的 Cache 资源浪费，会让连接到远程 PCIe 总线上的设备（不同 NUMA 节点间通信）作业性能急剧下降。

由于这个特性，基于 NUMA 开发的应用程序应该尽可能避免跨节点的远程内存访问。因为，跨节点内存访问不仅通信速度慢，还可能需要处理不同节点间内存和缓存的数据一致性。多线程在不同节点间的切换，是需要花费大成本的。

虽然 NUMA 相比于 SMP 具有更好的处理器扩展性，但因为 NUMA 没有实现彻底的主存隔离。所以 NUMA 远没有达到无限扩展的水平，最多可支持几百个 CPU。这是为了追求更高的并发性能所作出的妥协，一个节点未必就能完全满足多并发需求，多节点间线程切换实属一个折中的方案。这种做法使得 NUMA 具有一定的伸缩性，更加适合应用在服务器端。

### 1.3、MPP大规模并行处理结构

**MPP（Massive Parallel Processing，大规模并行处理）**，既然 NUMA 扩展性的限制是没有完全实现资源（e.g. 存储器、互联模块）的隔离性，那么 MPP 的解决思路就是为处理器提供彻底的独立资源。

![](../.gitbook/assets/image%20%2841%29.png)

MPP 拥有多个真正意义上的独立的 SMP 单元，每个 SMP 单元独占并只会访问自己本地的内存、I/O 等资源，SMP 单元间通过节点互联网络进行连接（Data Redistribution，数据重分配），是一个完全无共享（Share Nothing）的 CPU 计算平台结构。

![](../.gitbook/assets/image%20%2855%29.png)

MPP 的典型特征就是\*\*「多 SMP 单元组成，单元之间完全无共享」\*\*。除此之外，MPP 结构还具有以下特点：

* 每个 SMP 单元内都可以包含一个操作系统副本，所以每个 SMP 单元都可以运行自己的操作系统
* MPP 需要一种复杂的机制来调度和平衡各个节点的负载和并行处理过程，目前一些基于 MPP 技术的服务器往往通过系统级软件（e.g. 数据库）来屏蔽这种复杂性
* MPP 架构的局部区域内存的访存延迟低于远地内存访存延迟，因此 Linux 会自定采用局部节点分配策略，当一个任务请求分配内存时，首先在处理器自身节点内寻找空闲页，如果没有则到相邻的节点寻找空闲页，如果还没有再到远地节点中寻找空闲页，在操作系统层面就实现了访存性能优化

因为完全的资源隔离特性，所以 MPP 的扩展性是最好的，理论上其扩展无限制，目前的技术可实现 512 个节点互联，数千个 CPU，多应用于大型机。

## 2、Linux上的NUMA

### 2.1、基本概念

* **Node**：包含有若干个 Socket 的组
* **Socket**：表示一颗物理 CPU 的封装，简称插槽。Intel 为了避免将物理处理器和逻辑处理器混淆，所以将物理处理器统称为插槽。
* **Core**：Socket 内含有的物理核。
* **Thread**：在具有 Intel 超线程技术的处理器上，每个 Core 可以被虚拟为若干个（通常为两个）逻辑处理器，逻辑处理器会共享大多数内核资源（e.g. 内存缓存、功能单元）。逻辑处理器被统称为 Thread。
* **Processor**：处理器的统称，可以区分为物理处理器（physical processor）和逻辑处理器（virtual processors），对于大多数应用程序而言，它们并不关心处理器是物理的还是逻辑的。
* **Siblings**：表示每一个 physical processor 所含有的 virtual processors 的数量。（如果Siblings的数量等于Core数量，说明没有开启超线程 ）

![](../.gitbook/assets/image%20%2843%29.png)

**包含关系**：`NUMA Node > Socket > Core > Thread`

![](../.gitbook/assets/image%20%286%29.png)

**EXAMPLE**：上图为一个 NUMA Topology，表示该服务器具有 2 个 numa node，每个 node 含有一个 socket，每个 socket 含有 6 个 core，每个 core 又被超线程为 2 个 thread，所以服务器总共的 processor = 2\*1\*6\*2 = 24 颗。

### 2.2、NUMA调度策略

Linux 的每个进程或线程都会延续父进程的 NUMA 策略，优先会将其约束在一个 NUMA node 内。当然了，如果 NUMA 策略允许的话，进程也可以调用其他 node 上的资源。

**NUMA 的 CPU 分配策略有下列两种：**

* cpunodebind：约束进程运行在指定的若干个 node 内
* physcpubind：约束进程运行在指定的若干个物理 CPU 上

**NUMA 的 Memory 分配策略有下列 4 种：**

* localalloc：约束进程只能请求访问本地内存
* preferred：宽松地为进程指定一个优先 node，如果优先 node 上没有足够的内存资源，那么进程允许尝试运行在别的 node 内
* membind：规定进程只能从指定的若干个 node 上请求访问内存，并不严格规定只能访问本地内存
* interleave：规定进程可以使用 RR 算法轮转地从指定的若干个 node 上请求访问内存

### 2.3、获取宿主机的NUMA拓扑

下方两个脚本都可以使用

```bash
#!/bin/bash

function get_nr_processor()
{
   grep '^processor' /proc/cpuinfo | wc -l
}

function get_nr_socket()
{
   grep 'physical id' /proc/cpuinfo | awk -F: '{
           print $2 | "sort -un"}' | wc -l
}

function get_nr_siblings()
{
   grep 'siblings' /proc/cpuinfo | awk -F: '{
           print $2 | "sort -un"}'
}

function get_nr_cores_of_socket()
{
   grep 'cpu cores' /proc/cpuinfo | awk -F: '{
           print $2 | "sort -un"}'
}

echo '===== CPU Topology Table ====='
echo

echo '+--------------+---------+-----------+'
echo '| Processor ID | Core ID | Socket ID |'
echo '+--------------+---------+-----------+'

while read line; do
   if [ -z "$line" ]; then
       printf '| %-12s | %-7s | %-9s |\n' $p_id $c_id $s_id
       echo '+--------------+---------+-----------+'
       continue
   fi

   if echo "$line" | grep -q "^processor"; then
       p_id=`echo "$line" | awk -F: '{print $2}' | tr -d ' '`
   fi

   if echo "$line" | grep -q "^core id"; then
       c_id=`echo "$line" | awk -F: '{print $2}' | tr -d ' '`
   fi

   if echo "$line" | grep -q "^physical id"; then
       s_id=`echo "$line" | awk -F: '{print $2}' | tr -d ' '`
   fi
done < /proc/cpuinfo

echo

awk -F: '{
   if ($1 ~ /processor/) {
       gsub(/ /,"",$2);
       p_id=$2;
   } else if ($1 ~ /physical id/){
       gsub(/ /,"",$2);
       s_id=$2;
       arr[s_id]=arr[s_id] " " p_id
   }
}

END{
   for (i in arr)
       printf "Socket %s:%s\n", i, arr[i];
}' /proc/cpuinfo

echo
echo '===== CPU Info Summary ====='
echo

nr_processor=`get_nr_processor`
echo "Logical processors: $nr_processor"

nr_socket=`get_nr_socket`
echo "Physical socket: $nr_socket"

nr_siblings=`get_nr_siblings`
echo "Siblings in one socket: $nr_siblings"

nr_cores=`get_nr_cores_of_socket`
echo "Cores in one socket: $nr_cores"

let nr_cores*=nr_socket
echo "Cores in total: $nr_cores"

if [ "$nr_cores" = "$nr_processor" ]; then
   echo "Hyper-Threading: off"
else
   echo "Hyper-Threading: on"
fi

echo
echo '===== END ====='
```



```python
#!/usr/bin/env python
# SPDX-License-Identifier: BSD-3-Clause
# Copyright(c) 2010-2014 Intel Corporation
# Copyright(c) 2017 Cavium, Inc. All rights reserved.

from __future__ import print_function
import sys
try:
    xrange # Python 2
except NameError:
    xrange = range # Python 3

sockets = []
cores = []
core_map = {}
base_path = "/sys/devices/system/cpu"
fd = open("{}/kernel_max".format(base_path))
max_cpus = int(fd.read())
fd.close()
for cpu in xrange(max_cpus + 1):
    try:
        fd = open("{}/cpu{}/topology/core_id".format(base_path, cpu))
    except IOError:
        continue
    except:
        break
    core = int(fd.read())
    fd.close()
    fd = open("{}/cpu{}/topology/physical_package_id".format(base_path, cpu))
    socket = int(fd.read())
    fd.close()
    if core not in cores:
        cores.append(core)
    if socket not in sockets:
        sockets.append(socket)
    key = (socket, core)
    if key not in core_map:
        core_map[key] = []
    core_map[key].append(cpu)

print(format("=" * (47 + len(base_path))))
print("Core and Socket Information (as reported by '{}')".format(base_path))
print("{}\n".format("=" * (47 + len(base_path))))
print("cores = ", cores)
print("sockets = ", sockets)
print("")

max_processor_len = len(str(len(cores) * len(sockets) * 2 - 1))
max_thread_count = len(list(core_map.values())[0])
max_core_map_len = (max_processor_len * max_thread_count)  \
                      + len(", ") * (max_thread_count - 1) \
                      + len('[]') + len('Socket ')
max_core_id_len = len(str(max(cores)))

output = " ".ljust(max_core_id_len + len('Core '))
for s in sockets:
    output += " Socket %s" % str(s).ljust(max_core_map_len - len('Socket '))
print(output)

output = " ".ljust(max_core_id_len + len('Core '))
for s in sockets:
    output += " --------".ljust(max_core_map_len)
    output += " "
print(output)

for c in cores:
    output = "Core %s" % str(c).ljust(max_core_id_len)
    for s in sockets:
        if (s,c) in core_map:
            output += " " + str(core_map[(s, c)]).ljust(max_core_map_len)
        else:
            output += " " * (max_core_map_len + 1)
    print(output)
```

### 2.4、NUMA启动服务

```python
#启动mongodb时，指定NUMA策略。
numactl --interleave=all ${MONGODB_HOME}/bin/mongod \
--config conf/mongodb.conf
```

```text
#启动其他进程时，将进程启动命令，跟在numactl 之后

#python在node0中执行
numactl --cpubind=0 --membind=0 python param

#java在node1中执行    
numactl --cpubind=1 --membind=1 java param    
```

{% hint style="info" %}
内核参数zone\_reclaim\_mode，对NUMA 的影响

可选值0、1

当某个节点可用内存不足时：

0）如果为0的话，那么系统会倾向于从其他节点分配内存

1）如果为1的话，那么系统会倾向于从本地节点回收Cache内存多数时候

Cache对性能很重要，所以0是一个更好的选择
{% endhint %}

## 3、Nova实现NUMA亲和

除了上文中提到的 NUMA 基本概念之外，Nova 还自定义一些对象概念：

* **Cell**：NUMA Node 的通名词，供 Libvirt API 使用
* **vCPU**：虚拟机的 CPU，根据虚拟机 NUMA 拓扑的不同，一个虚拟机 CPU 可以是一个 socket、core 或 thread。
* **pCPU**：宿主机的 CPU，根据宿主机 NUMA 拓扑的不同，一个物理机 CPU 同样可以是一个 socket、core 或 thread。
* **Siblings Thread**：兄弟线程，即由同一个 Core 超线程出来的 Threads。
* **Host NUMA Topology**：宿主机的 NUMA 拓扑。
* **Guest NUMA Topology**：虚拟机的 NUMA 拓扑。

**NOTE 1**：vCPU 和 pCPU 的定义具有一定的迷惑性，简单来理解：虚拟机实际是宿主机的一个进程，虚拟机 CPU 实际是宿主机进程中的一个特殊的线程。引入 pCPU 和 vCPU 的概念是为了让上层逻辑能够屏蔽机器 NUMA 拓扑的复杂性。

**NOTE 2**：Thread siblings 对象的引入是为了无论服务器是否开启了超线程，Nova 同样能够支持物理 CPU 绑定的功能。

### 3.1、配置Nova使用NUMA亲和

1）**nova-scheduler 启用 NUMATopologyFilter**

```python
# nova.conf
[DEFAULT]
...
scheduler_default_filters=...,NUMATopologyFilter
```

2）设置flavor 属性

```text
openstack flavor set FLAVOR-NAME \
    --property hw:numa_nodes=FLAVOR-NODES \
    --property hw:numa_cpus.N=FLAVOR-CORES \
    --property hw:numa_mem.N=FLAVOR-MEMORY
```

* **FLAVOR-NODES（整数）**：设定 Guest NUMA nodes 的个数。如果不指定，则 Guest vCPUs 可以在任意可用的 Host NUMA nodes 上浮动。
* **N**：整数，Guest NUMA nodes ID，取值范围在 \[0, FLAVOR-NODES-1\]。
* **FLAVOR-CORES（逗号分隔的整数）**：设定分配到 Guest NUMA node N 上运行的 vCPUs 列表。如果不指定，vCPUs 在 Guest NUMA nodes 之间平均分配。
* **FLAVOR-MEMORY（整数）**：单位 MB，设定分配到 Guest NUMA node N 上 Memory Size。如果不指定，Memory 在 Guest NUMA nodes 之间平均分配。

**设定 Guest NUMA Topology 的两种方式：**

* **自动设定 Guest NUMA Topology**：仅仅需要指定 Guest NUMA nodes 的个数，然后由 Nova 根据 Flavor 设定的虚拟机规格平均将 vCPU/Mem 分布到不同的 Host NUMA nodes 上（默认从 Host NUMA node 0 开始分配）。

**NOTE**：选择使用自动设定方式时，建议一同使用 `hw:numa_mempolicy` 属性，表示 NUMA 的 Mem 访问策略，有严格访问本地内存的 strict 和宽松的 preferred 两种选择，这样可以最大程度降低配置参数的复杂性。而且对于某些特定工作负载的 NUMA 架构问题，比如：MySQL “swap insanity” 问题 ，或许 preferred 会是一个不错的选择。

* **手动设定 Guest NUMA Topology**：不仅指定 Guest NUMA nodes 的个数，还需要通过 `hw:numa_cpus.N` 和 `hw:numa_mem.N` 来指定每个 Guest NUMA nodes 上分配的 vCPUs 和 Memory Size。

Nova Scheduler 会根据参数 `hw:numa_nodes` 来决定如何映射 Guest NUMA node。如果没有设置该参数，那么 Scheduler 将自由的决定在哪里运行虚拟机，而无需关心单个 NUMA 节点是否能够满足虚拟机 flavor 中的 vCPU/Mem 配置，但仍会优先考虑选出一个 NUMA 节点就可以满足情况的计算节点。

* 如果 numa\_nodes = 1，Scheduler 将会选择出单个 NUMA 节点能够满足虚拟机 flavor 配置的计算节点。
* 如果 numa\_nodes &gt; 1，Scheduler 将会选择出 NUMA 节点数量以及 NUMA 节点中资源情况能够满足虚拟机 flavor 配置的计算节点。

**NOTE 1**：只有在设定了 `hw:numa_nodes` 后，`hw:numa_cpus.N` 和 `hw:numa_mem.N` 才会生效。只有当 Guest NUMA nodes 存在非对称访问 vCPU/Mem 时（Guest NUMA Nodes 之间拥有的 vCPU 数量和 Mem 大小并非是镜像的），才需要去设定这些参数。

**NOTE 2**：N 仅仅是 Guest NUMA node 的索引，并非实际上的 Host NUMA node 的 ID。例如，Guest NUMA node 0，可能会被映射到 Host NUMA node 1。类似的，FLAVOR-CORES 的值也仅仅是 vCPU 的索引。因此，Nova 的 NUMA 特性并不能用来约束 Guest vCPU/Mem 绑定到指定的 Host NUMA node 上。要完成 vCPU 绑定到指定的 pCPU，需要借助 CPU Pinning policy 机制。

**WARNING**：如果 `hw:numa_cpus.N` 和 `hw:numa_mem.N` 的设定值大于虚拟机本身可用的 CPUs/Mem，则触发异常。

**EXAMPLE**：定义虚拟机有 4 vCPU，4096MB Mem，设定 Guest NUMA topology 为 2 Guest NUMA node：

* Guest NUMA node 0：vCPU 0、Mem 1024MB
* Guest NUMA node 1：vCPU 1/2/3、Mem 3072MB

```text
openstack flavor set aze-FLAVOR \ 
  --property hw:numa_nodes=2 \ 
  --property hw:numa_cpus.0=0 \ 
  --property hw:numa_cpus.1=1,2,3 \ 
  --property hw:numa_mem.0=1024 \ 
  --property hw:numa_mem.1=3072 \
```

NOTE：numa\_cpus 指定的是 vCPUs 的序号，而非 pCPUs。

使用该 flavor 创建的虚拟机时，最后由 Libvirt Driver 完成将 Guest NUMA node 映射到 Host NUMA node 上。

{% hint style="danger" %}
建议加上 numa\_cpus 和 numa\_mem 参数，手动将flavor 资源平均分配到NUMA Node 中
{% endhint %}

3）设置image 属性

除了通过 Flavor extra-specs 来设定 Guest NUMA topology 之外，还可以通过 Image Metadata 来设定。e.g.

```text
glance image-update --property \ 
    hw_numa_nodes=2 \ 
    hw_numa_cpus.0=0 \ 
    hw_numa_mem.0=1024 \ 
    hw_numa_cpus.1=1,2,3 \ 
    hw_numa_mem.1=3072 \ 
    image_name
```

注意，当镜像的 NUMA 约束与 Flavor 的 NUMA 约束冲突时，以 Flavor 为准。

4）查看虚机cpu的绑定情况

```text
# virsh vcpuinfo instance-0000000c
VCPU:           0
CPU:            2
State:          running
CPU time:       3.0s
CPU Affinity:   --y-----

VCPU:           1
CPU:            5
State:          running
CPU time:       1.0s
CPU Affinity:   -----y--
```

## 4、Numa命令

```text
# numactl -H
available: 1 nodes (0)
node 0 cpus: 0 1 2 3
node 0 size: 8191 MB
node 0 free: 156 MB
node distances:
node   0
  0:  10
```

```text
# numastat -m -c

Per-node system memory usage (in MBs):
                 Node 0 Total
                 ------ -----
MemTotal           8191  8191
MemFree             157   157
MemUsed            8035  8035
Active             6230  6230
Inactive            866   866
Active(anon)       5229  5229
Inactive(anon)      134   134
Active(file)       1000  1000
Inactive(file)      732   732
Unevictable          75    75
Mlocked              75    75
Dirty                 0     0
Writeback             0     0
FilePages          2137  2137
Mapped               74    74
AnonPages          5034  5034
Shmem               386   386
KernelStack          11    11
PageTables           47    47
NFS_Unstable          0     0
Bounce                0     0
WritebackTmp          0     0
Slab                276   276
SReclaimable        202   202
SUnreclaim           73    73
AnonHugePages       798   798
HugePages_Total       0     0
HugePages_Free        0     0
HugePages_Surp        0     0
```

## 5、文档链接

[openstack-CPU topologies](https://docs.openstack.org/nova/pike/admin/cpu-topologies.html#customizing-instance-numa-placement-policies)

[numastat命令](https://blog.51cto.com/hl914/1557615?source=drt)

[OpenStack Nova 高性能虚拟机之 NUMA 架构亲和](https://www.cnblogs.com/jmilkfan-fanguiju/p/10589768.html#Nova__NUMA__408)

[NUMA特性对MySQL性能的影响](https://cloud.tencent.com/developer/article/1159058)

[redhat-在openstack中配置NUMA](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux_openstack_platform/7/html/instances_and_images_guide/ch-cpu_pinning)




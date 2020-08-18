# 热迁移配置参数

热迁移，也叫在线迁移（live migration），是虚拟机从一台主机上迁移到另一台主机上但是“不中断VM服务”的一种机制。这里“不中断”指的是在用户观感上，VM一直提供服务。而实际上，在热迁移的过程中，VM会短暂地暂停一段时间。

openstack的热迁移是nova组件调用libvirtd完成的。我们一般通过nova配置文件（path：/etc/nova/nova.conf）来设置有关热迁移的参数，从而让VM按照我们希望的方式进行在线迁移。

nova配置文件有关热迁移的参数有如下几个：

* live\_migration\_bandwidth
* live\_migration\_downtime
* live\_migration\_downtime\_steps
* live\_migration\_downtime\_delay
* live\_migration\_completion\_timeout
* live\_migration\_permit\_post\_copy
* live\_migration\_permit\_auto\_converge 

下面详细地介绍这几个参数代表的意义以及对热迁移产生的影响。

> **live\_migration\_bandwidth**
>
> 它是迁移期间要使用的最大带宽（以MiB / s为单位），默认值是0，意味着不限制迁移带宽。

> **live\_migration\_downtime、live\_migration\_downtime\_steps、live\_migration\_downtime\_delay**
>
> 这三个参数都是有关热迁移停机时间（downtime）的。我们说热迁移过程中VM会短暂的暂停，而VM暂停的时间就是停机时间，它与这三个参数有关。
>
> 实际上，热迁移是转移内存（或存储）的过程。源主机不断把虚拟机的内存转移到目的主机，直到源主机仅仅省一部分可以一次转移完成的内存未被转移，此时把源主机上的虚拟机暂停，转移掉最后这一部分。这样做是因为，在转移内存时，源主机上的虚拟机有可能在提供有关内存写入的服务，生成新的未被迁移的内存，这就是所谓的“脏内存”，如果不暂停虚拟机，“脏内存”一直产生，迁移永远不能完成。
>
> openstack下热迁移的停机时间（downtime）不是单纯设置一个数字那么简单，它停机策略要稍微复杂一点。openstack在迁移过程中的停机时间是变化的，该值不断增加，在一段时间后停机时间达到最终的最大值，这个值就是由**live\_migration\_downtime**设定的。而**live\_migration\_downtime\_steps**表示达到最大值所经历的次数，**live\_migration\_downtime\_delay**表示每一次增加downtime的间隔时间。下面举一个实验的例子，来具体说明停机时间是如何变化的：
>
> 进行实验的虚拟机的内存为4GiB。在/etc/nova/nova.conf设置的三个参数的值如下：
>
> live\_migration\_downtime=5000 / live\_migration\_downtime\_steps=7 / live\_migration\_downtime\_steps=75
>
> 查看nova的log日志（path：/var/log/nova/nova-compute.log），找到了nova的变化情况
>
> 1. Increasing downtime to 714 ms after 0 sec elapsed time
> 2. Increasing downtime to 1326 ms after 300 sec elapsed time
> 3. Increasing downtime to 1938 ms after 600 sec elapsed time
> 4. Increasing downtime to 2550 ms after 900 sec elapsed time
> 5. Increasing downtime to 3162 ms after 1200 sec elapsed time
> 6. Increasing downtime to 3774 ms after 1500 sec elapsed time
> 7. Increasing downtime to 4386 ms after 1800 sec elapsed time
> 8. Increasing downtime to 4998 ms after 2100 sec elapsed time
>
> 可以看到，初始的downtime是一个较小的值，经历七步以后，downtime增加到接近5000（ms）。初始的downtime值是由**live\_migration\_downtime**/**live\_migration\_downtime\_steps**得来的，这里就是5000/7=714（ms）。每一步增加的时间是由初始的downtime\*\(**live\_migration\_downtime\_steps-1**\)**/live\_migration\_downtime\_steps**得到的，这里就是714\*\(7-1\)/7=612（ms）。
>
> 除此之外，我们发现，迁移每进行300s，downtime就变化一次，300s这个时间是由**live\_migration\_downtime\_delay**决定的。在这里，300s=**live\_migration\_downtime\_delay**\*VM内存大小（GiB），即75\*4=300。
>
> 在实际实际工作中，在最大停机时间一定的情况下，有时我们需要尽快完成迁移，不在乎停机的时间，这时我们减少**live\_migration\_downtime\_steps**和**live\_migration\_downtime\_delay**，尽快达到最大停机时间；有时我们不在乎迁移总时长，希望服务中断的时间越短越好， 就尽量增大**live\_migration\_downtime\_steps**和**live\_migration\_downtime\_delay**，慢慢地达到最大停机时间，这样在有可能在最大停机时间内完成VM的迁移。

> **live\_migration\_completion\_timeout**
>
> 这个参数表示整个迁移过程所允许的最大迁移时间，若超过这个时间迁移未完成，则迁移失败，取消迁移。这个参数跟**live\_migration\_downtime\_delay**一样，也与内存有关系。实际的最大完成时间=**live\_migration\_completion\_timeout**\*VM内存大小（GiB）。**live\_migration\_completion\_timeout**的默认值为800，假如虚拟机的内存大小是4GiB，那么意味着在3200s的时候，未迁移完成的该虚拟机停止迁移，迁移失败。

> **live\_migration\_permit\_post\_copy**
>
> **live\_migration\_permit\_post\_copy**是一种迁移模式，叫做后拷贝。在不设定此模式的情况下，VM都是通过前拷贝完成的，所谓前拷贝指的是VM上所有内存数据都是在切换到目标主机之前拷贝完的。而后拷贝会先尽可能快的切换到目标主机，因此后拷贝模式会先传输设备的状态和一部分\(10%\)数据到目标主机，然后就简单的切换到目标主机上面运行虚拟机。当发现访问虚拟机的某些内存page不存在时，就会产生一个远程页错误，触发从源主机上面拉取该page的动作。这种后拷贝的迁移模式相对于前拷贝更加危险，当网络不同，或任意一台主机出现故障的时候，虚拟机会出现异常。

> **live\_migration\_permit\_auto\_converge**
>
> **live\_migration\_permit\_auto\_converge**是另一种迁移模式，叫做自动收敛。它在虚拟机长时间处于高业务下而影响迁移的时候，调整vcpu的参数减少vcpu的负载，降低“脏内存”的增长速度，从而使迁移能够顺利完成。
>
> 在libvirtd中，当“脏内存“的增长速度大于迁移内存传输速度时，触发自动收敛模式，libvirtd在一开始降低了20%的vcpu性能，如果“脏内存“的增长速度依旧大于迁移内存传输速度，则在此基础上再降低10%的性能，直到“脏内存“的增长速度小于迁移内存传输速度。值得注意的是，该模式只能保证虚拟机未迁移的内存持续减少，但不能保证迁移在设定的最大迁移时间内迁移完成。


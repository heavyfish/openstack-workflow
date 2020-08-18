# 故障根磁盘挂载修复



1、关闭故障的虚拟机。

2、手动创建 故障根盘的 xml

![](../.gitbook/assets/image%20%2861%29.png)

实际的根盘 xml

```text
<disk type='network' device='disk'>
      <driver name='qemu' type='raw' cache='writeback' discard='unmap'/>
      <auth username='cinder'>
        <secret type='ceph' uuid='8d2eb2bb-2bac-4d2f-9726-83029caef708'/>
      </auth>
      <source protocol='rbd' name='volumes/volume-4e81e249-11dc-47b2-b016-20dd85b3b7b9'>
        <host name='172.16.80.11' port='6789'/>
        <host name='172.16.80.12' port='6789'/>
        <host name='172.16.80.14' port='6789'/>
      </source>
      <target dev='vdc' bus='virtio'/>
</disk>
```

3、将故障虚拟机根硬盘通过attach挂载到恢复用的虚拟机

```text
virsh attach-device <instance-新虚机> <根盘.xml>
```

4、登录恢复用虚拟机，执行fsck

5、通过detach 解绑

```text
virsh detach-device <instance-新虚机> <根盘.xml>
```

6、虚拟机启动


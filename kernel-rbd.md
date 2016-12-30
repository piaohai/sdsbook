Kernel-rbd，通过内核模块rbd.ko （块设备协议对接）与 libceph.ko （RADOS协议对接，与集群通信），向外提供一个虚拟的块设备（如：/dev/rbd0）。用户可以向使用传统块设备（如：/dev/sda）一样使用/dev/rbd0，对其进行分区，格式化，挂载等。

![](/assets/kernel_rbd_1.png)

Kernel RBD 使用方法，请参见：[RBD 使用手册](http://docs.ceph.org.cn/man/8/rbd/)


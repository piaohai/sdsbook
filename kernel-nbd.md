Kernel-nbd，通过 nbd.ko 内核模块，将一个块设备映射为网络块设备。区别于 Kernel RBD ，网络块块设备与后端的交互逻辑是在用户空间实现了，Ceph 通过 rbd-nbd 实现了用户空间 NBDServer 的逻辑。这样做的好处是，在主机上使用块设备不用在依赖于 rbd.ko 内核模块了，该内核模块代码逻辑需要在内核中实现，这将导致一旦内核模块出现 BUG，我们将很难定位和修复，另外由于 Ceph 目前发展速度快，而内核代码的合并是十分谨慎的，故很多 Ceph 中基于块设备的新特性都无法通过 Kernel RBD 的方式被使用。

关于 Kernel-nbd 的使用，请参见：[rbd-nbd 使用手册](http://docs.ceph.org.cn/man/8/rbd-nbd/)


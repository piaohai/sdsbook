RBD FUSE 为 RBD 块设备在用户空间的一种实现，依赖于 FileSystem FUSE 内核模块，这样的好处是在没有 rbd.ko 或者 nbd.ko 的内核模块时，可以通过 RBD FUSE 模式将块设备映射至主机。然而由于这种映射是文件系统用户空间的一种实现，RBD 块设备只能映射至主机某个文件夹下的一个文件而已。如果用户想要向使用块设备一样使用它，就必须将该文件通过 loopback device 映射为本地 loop 块设备。这样做将严重的影响使用块设备时的性能与稳定性 —— 尤其对于高 IOPS，高并发场景。在没有 NBD 访问方式时，通常使用 RBD FUSE 访问方式来测试 RBD 块设备的新特性，该使用方式现在已逐步的被废弃。

关于 rbd-fuse 的使用，请参见：[rbd-fuse手册](http://docs.ceph.org.cn/man/8/rbd-fuse/)


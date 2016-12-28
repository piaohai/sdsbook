Qemu-rbd，通常情况下与虚拟化结合使用，为虚拟机提供块存储服务。Qemu-rbd在Qemu中基于librados，封装出了对虚拟块设备的访问方式，使得Qemu进程能够通过原生virtio驱动，直接对接librados，实现对集群的访问。

![](/assets/qemu_rbd_1.png)

Qemu RBD的使用，请参见：[Qemu RBD使用手册](http://docs.ceph.org.cn/rbd/qemu-rbd/)，[Libvirt使用手册](http://docs.ceph.org.cn/rbd/libvirt/)


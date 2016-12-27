## Ceph MDS进程

非必需的角色，只有集群对外提供CephFS服务时，需要部署并启动MDS进程。MDS进程管理并缓存CephFS文件系统的目录树结构，加速对元数据的访问速度，并将元数据持久化入指定的存储资源池（Pool）中。

![](/assets/mds_1.png)


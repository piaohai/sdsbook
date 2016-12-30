## Ceph MON进程

负责存储集群全局视图cluster map（包括MONmap，OSDMap，MDSMap，PGMap等），管理和更新集群的状态，让Client在任何时刻观察集群，都能够获得其唯一的集群视图。另外，Monitor还存储并管理一些权限认证Auth的信息，部分重要的MON，OSD，MDS日志也会存在Monitor中，以便于通过客户端直接查询集群中所有角色的关键日志信息。

![](/assets/mon_1.png)


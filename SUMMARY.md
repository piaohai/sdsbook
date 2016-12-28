# Summary

* [Introduction](README.md)
* [传统存储技术](传统存储技术.md)
    * [外接存储](外接存储.md)
        * [Direct-attached Storage —— DAS](direct-attached-storage-——-das.md)
        * [Network-attached Storage —— NAS](network-attached-storage-——-nas.md)
        * [Storage Area Network —— SAN](storage-area-network-——-san.md)
    * [存储网络与协议](网络与协议.md)
        * [块存储](块存储.md)
            * [FC](fc网络.md)
            * FCoE
            * FCIP
            * ISCSI
    * [数据冗余](数据冗余.md)
        * [RAID技术](raid.md)
        * [拷贝技术——Replication](replication.md)
        * [纠删码技术——Erasure Code](纠删码技术——erasure-code.md)
        * 数据网关
        * 高级特性
* [Ceph分布式存储](ceph分布式存储.md)
    * [架构概览](架构概览.md)
    * [Ceph客户端](ceph客户端.md)
        * [Native Protocol —— Librados](base.md)
        * [Up Service](up-service.md)
            * [块存储](块存储.md)
            * 对象存储
            * [文件服务](文件服务.md)
    * [Ceph集群](ceph集群.md)
        * [进程角色](daemon角色.md)
            * [MON](mon.md)
            * [OSD](osd.md)
            * [MDS](mds.md)
        * [核心概念](核心概念.md)
            * [Placement Group](placement-group.md)
            * [Pool](pool.md)
            * [CRUSH](crush.md)
            * [Cluster Map](cluster-map.md)
            * [IO Mapping](io-mapping.md)


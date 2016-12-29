RAID技术——Redundant array of independent disks，是一种早期的存储虚拟化技术，它可以将多块物理磁盘组织成一块逻辑磁盘，并且让这块逻辑磁盘能够具有更高的性能，更高的可靠性。当前主流的 RAID 技术有：

* RAID0：条带化技术，将数据条带化后，存储与多个物理磁盘上，提升逻辑磁盘的并发性能与整体容量。

  ![](/assets/raid_1_1.png)

* RAID1：数据镜像，通常使用两个物理磁盘组合成为一个虚拟磁盘，向虚拟磁盘写入数据同时保存至这两个物理磁盘，从而保证其中一个磁盘损坏后，数据仍然可用。

  ![](/assets/raid_2_1.png)

* RAID2：现在已经

  ![](/assets/raid_3.png)

* RAID3

* RAID4

* RAID5

* RAID6

上诉的RAID技术可以组合使用，通常的组合有：

* RAID01
* RAID03
* RAID10
* RAID50
* RAID60
* RAID100




RAID技术——Redundant array of independent disks，是一种早期的存储虚拟化技术，它可以将多块物理磁盘组织成一块逻辑磁盘，并且让这块逻辑磁盘能够具有更高的性能，更高的可靠性。当前主流的 RAID 技术有：

* RAID0：条带化技术，将数据条带化后，存储与多个物理磁盘上，提升逻辑磁盘的并发性能与整体容量。

  ![](/assets/raid_1_1.png)

* RAID1：数据镜像，通常使用两个物理磁盘组合成为一个虚拟磁盘，向虚拟磁盘写入数据同时保存至这两个物理磁盘，从而保证其中一个磁盘损坏后，数据仍然可用。

  ![](/assets/raid_2_1.png)

* RAID2：现在已经被废弃了，RAID2使用hamming码对bit进行校验计算，而不是基于block的，这对于磁盘这种块设备来说非常不适合

  ![](/assets/raid_3.png)

* RAID3：现在已经被废弃了，RAID3基于RAID2的改善是，校验计算从bit改进到了byte，基于byte计算校验，而这仍然不适合磁盘

![](/assets/raid_4.png)

* RAID4：现在已经被废弃了，RAID4基于RAID3的改善是，检验计算从byte改进到了block，基于block级别进行计算校验，这对于磁盘随机读写来说，已经大大提升了磁盘的性能。

![](/assets/raid_5.png)

* RAID5：RAID4 仍然存在严重的问题，就是在数据更新时候，涉及任何磁盘的block更新，都需要同时改变parity，而parity都存储在同一块磁盘上面，这导致了写入更新的性能瓶颈完全在于 parity disk 的性能带宽。为了解决这个问题，RAID5 采用了将parity均匀的分布在4块磁盘之间，从而使其不出现性能瓶颈。

![](/assets/raid_6.png)

* RAID6：RAID5 只使用一块磁盘的容量作为 parity，如果出现两块磁盘损坏的情况，数据就会出现丢失。而实际上，RAID 在数据重建的过程中，会对磁盘进行全盘扫描读写，这种场景下磁盘的损坏概率会增大。为了保证在某些数据可靠性要求较高的场景下，在损坏两个磁盘后数据能够不丢失，故产生了 RAID6。RAID6 相对于 RAID5 的唯一区别是，RAID6 使用了两块磁盘容量作为 parity，能够容忍两块磁盘同时损坏时数据不丢失。

![](/assets/raid_7.png)

由于上面介绍的 RAID 技术，对磁盘数量要求是有限制的，比如 RAID5，只能支持4块磁盘为一组，RAID6 只能支持5块磁盘为一组。那么如果一个服务上面有20块磁盘，这时候怎么办呢？在这种情况下，RAID 组合使用的用法产生了，通常的组合方式有：

* RAID01：RAID0 ＋ RAID1fd 

![](/assets/raid_8.png)
* RAID03：RAID0 ＋ RAID3

![](/assets/raid_9.png)
* RAID10：RAID1 ＋ RAID0

![](/assets/raid_10.png)

* RAID50：RAID5 ＋ RAID0

![](/assets/raid_11.png)

* RAID60：RAID6 ＋ RAID0

![](/assets/raid_12.png)

* RAID100：RAID1 + RAID0 + RAID0

![](/assets/raid_13.png)


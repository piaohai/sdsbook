在早期，由于数据量小，并发访问量小，应用的数量少，那时候数据存在服务器本地就能够满足需求 —— 在本地服务器将多块硬盘做成磁盘阵列，并且十分易于管理。

然而本地服务器的硬盘接口数量是有限的，随着企业的数据的不断增长，数据越来越多，服务器上的盘位已经消耗殆尽了，单台服务器对于存储容量的需求开始不能够被满足，此时运维IT人员通常会把服务器上容量较小的硬盘替换成容量较大的硬盘。但是随着企业的快速发展，数据增长数据越来越快，已经远远超过磁盘容量发展的速度，那怎么办呢？这时，Direct-attached storage（DAS）出现了，DAS可以将一个外部的磁盘阵列通过有线连接的方式，直接接入到一台服务器上，那么该服务器就能够访问到额外的存储资源了。

在我们生活中，PC电脑外接的硬盘盒就是一种DAS产品，通常可以通过USB接口或者eSATA接口与PC电脑进行连接。

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTcn9P7YUSD8VsFbkkgm75QfF87tZUsSz7d8PM0_0F_P50h53z5jw "Image result for RAID 硬盘盒")

在企业中，DAS通常是一个磁盘阵列服务器，通过FC，SCSI，ATA线缆与客户端服务器直连。

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTNxnpVi3Nq9iywsb6Et5p79Ht6CzD-RmdgwAeiXwNMhlMMdOmDTg "Image result for DAS 存储")


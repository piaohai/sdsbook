CRUSH作为Ceph集群最为核心的数据分布算法，需要解决大规模分布式存储系统面临艰巨的任务。当前存储设备已经发展至数十、数百、甚至是数千的存储设备，而数据通常会达到PB这个数量级以至更高。当前的存储系统必须能够均匀的分配数据和工作负载，以高效的利用现有可用资源、系统性能， 同时要便于集群的扩展以及对硬件故障的处理。  
CRUSH算法正是实现了一个伪随机的数据分发功能，它被设计用于基于对象的分布式存储系统，这样的系统不依赖于某个中央目录就能够实现数据对象到存储设备的映射。另外大型存储系统的发展本质就是动态的，所以CRUSH被设计成便于增加和移除存储设备，同时能将非必要的数据移动降至最低。该算法还包括了各种各样的数据复制和可靠性机制，并按照用户定义的策略来分发数据，而且这样的策略还能强制执行跨越故障域的备份分离。

## CRUSH Map

CRUSH Map是用户为集群定义的一颗具有层次结构的树，如下图：

![](/assets/crush_3.png)

层次结构中包含只包含两种角色：

* Bucket（桶）：表示除 OSD 之外的角色
* Device（设备）：只有 OSD 才能是 Device 角色

在 CRUSH Map 中，叶子节点必须为 Device （即 OSD），非叶子节点必须为 Bucket。通常情况下，数据中心实际的物理拓扑来决定CRUSH Map，如上图示例，根节点下包含一个数据中心，该数据中心下有一个机房，该机房中有一个机架，该机架下有两台服务器，每台服务器下面有两个OSD。CRUSH Map 树结构能很好的来描述各个层级之间的包含关系，而 CRUSH 算法可以根据这个层级树以及定义好的 CRUSH RULE 来决定最终如何将 PG 落到 OSD 上。

当然 CRUSH MAP 某些时候还可以根据实际需求定义或划分为逻辑拓扑，比如：我们希望区分不同性能的存储设备

![](/assets/crush_2.png)

上图的拓扑可以定义如下CRUSH Map

```
$ sudo ceph osd tree
# id  weight  type name up/down reweight
-21 12  root ssd
-22 2       host ceph-osd2-ssd
6 1             osd.6 up  1
9 1             osd.9 up  1
-23 2       host ceph-osd1-ssd
8 1             osd.8 up  1
11  1           osd.11  up  1
-24 2       host ceph-osd0-ssd
7 1             osd.7 up  1
10  1           osd.10  up  1
-1  12  root sata
-2  2       host ceph-osd2-sata
0 1             osd.0 up  1
3 1             osd.3 up  1
-3  2       host ceph-osd1-sata
2 1             osd.2 up  1
5 1             osd.5 up  1
-4  2       host ceph-osd0-sata
1 1             osd.1 up  1
4 1             osd.4 up  1
```

CRUSH Map对数据安全也有重要影响，CRUSH 算法可以通过 CRUSH Map 决定数据的映射，从而避免物理故障导致多个副本同时失效的情况发生。比如：共用的电源，共用的网络，CRUSH 算法可以根据 CRUSH Map，将副本放置在不同故障域中（故障域通常为机架级，也可是机房级或主机级的），同时仍然在各个故障域中保持数据所需的分布性。

总之，CRUSH Map 是一个层级视图，描述了各个 OSD 在层级中所处的角色与位置。

## CRUSH RULES

CRUSH RULES 基于 CRUSH Map，定义了选择 Device（OSD） 的规则和步骤，一个典型的 CRUSH RULES：

```
rule rbd {
    ruleset 2
    type replicated
    min_size 0
    max_size 10
    step take root                        ··· 1)
    step chooseleaf firstn 0 type host    ··· 2)
    step emit                             ··· 3)
}
```

上面的规则描述了一个简单的选择步骤：

1. 选择根节点
2. 选择副本数量的叶子节点，并且叶子节点需要落在不同的主机中
3. 完成选择

下面是一个复杂一些的规则：

```
rule ssd-primary {
    ruleset 5
    type replicated
    min_size 5
    max_size 10
    step take ssd                           ··· 1)
    step chooseleaf firstn 1 type host   ··· 2)
    step emit                              ··· 3)
    step take sata                          ··· 4)
    step chooseleaf firstn -1 type host     ··· 5)
    step emit                              ··· 6)
}
```

上面的规则描述了的选择步骤如下：

1. 选择根节点ssd
2. 选择一个叶子节点，并且叶子节点需要落在不同的主机中
3. 完成选择第一次选择
4. 再次选择根节点sata
5. 选择（副本数量－1）个叶子节点，并且叶子节点需要落在不同的主机中
6. 完成选择第二次选择

该规则能够完成将主副本放置在ssd节点上，而将其它副本放置在sata节点上

总之，CRUSH RULE 定义了一个规则，该规则包含若干的步骤，而这些步骤直接决定了 CRUSH 算法按照什么样的步骤去选择一定数量的OSD，来存放PG。

## CRUSH Algorithm

CRUSH 算法的设置目的是使数据能够根据Device的权重，（Device的权重，即 OSD 的权重，是用户根据磁盘大小事先定义好的）均地分布，并在概率上，保持一个相对的平衡。副本放置在具有层次结构的存储设备中，能够根据 CRUSH RULE 对故障域的定义，将数据映射到特定的 OSD 上。

CRUSH 算法给定一个输入向量 **x**，输出一个确定的有序的储存目标向量 **R**。在 Ceph 中 **x** 为 （PGid, crush map, crush rules\)，当输**x**，CRUSH利用强大的多重整数hash函数根据集群map、定位规则、以及 **x **计算出确定的、独立的、可靠的映射关系。CRUSH分配算法是伪随机算法，并且输入的内容和输出的储存位置之间是没有显性相关的，所以我们可以说CRUSH 特定的算法在集群设备中生成了一组向量 **R**，指向副本应该存放的OSD（如**R** ＝［1，2，5］，表示 PG有三个拷贝，且应该被放置在osd.1, osd.2, osd.5上）。

CRUSH 算法描述了使用什么算法将 PG 映射至 OSD 的过程：

![](/assets/crush_4.png)


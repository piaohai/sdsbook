## 概念

PG（放置组） — Placement Group，当数据到存储集群时，对象首先被映射至PG，然后再映射至OSDs。通过这样的方式我们可以将存储对象进行分组，进而减少对每个对象的元数据跟踪与使用时的开销（每个对象其放置历史的跟踪成本其实相当高）。增加PGs的数量可以减少每个OSD在集群中负载的方差（使OSDs负载更加平衡），但是越多的PG会消耗更多的CPU和内存。一般我们建议保持每100 PGs/OSD，虽然设置太高或太低时都会对并发性能，可靠性，数据平衡度产生影响，而这还将取决于群集的规模大小。

![](/assets/pg_1.png)

## 作用

基本特点：

* PG 确定了 pool 中的对象和 OSD 之间的映射关系。一个 object 只会存在于一个 PG 中，但是多个 object 可以在同一个 PG 内。
* Pool 的 PG 数目是创建 pool 时候指定的，Ceph 官方有推荐的计算方法。其值与 OSD 的总数的关系密切。当Ceph 集群扩展 OSD 增多时，根据需要，可以增加 pool 的 PG 数目。
* 对象的副本数目，也就是被拷贝的次数，是在创建 Pool 时指定的。该分数决定了每个 PG 会在几个 OSD 上保存对象。如果一个拷贝型 Pool 的size（拷贝份数）为 2，它会包含指定数目的 PG，每个 PG 使用两个 OSD，其中，第一个为主 OSD （primary），其它的为从 OSD （secondary），不同的 PG 可能会共享一个 OSD。
*Ceph 引入 PG 的目的主要是为了减少直接将对象映射到 OSD 的复杂度。
*PG 也是Ceph 集群做清理（scrubbing）的基本单位，也就是说数据清理是一个一个PG来做的。
*PG 和 OSD 之间的映射关系由 CRUSH 决定，而它做决定的依据是 CRUSH 规则（rules）。CRUSH 将所有的存储设备（OSD）组织成一个分层结构，该结构能区分故障域（failure domain），该结构中每个节点都是一个 CRUSH bucket。详细情况请阅读 CRUSH 相关的文档。

PG 和 OSD 的关系是动态的：

*一开始在 PG 被创建的时候，MON 根据 CRUSH 算法计算出 PG 所在的 OSD。这是它们之间的初始关系。
*Ceph 集群中 OSD 的状态是不断变化的，它会在如下状态之间做切换：
*up：守护进程运行中，能够提供IO服务；
*down：守护进程不在运行，无法提供IO服务；
*in：包含数据；
*out：不包含数据；
*部分 PG 和 OSD 的关系会随着 OSD 状态的变化而发生变化。
*当新的 OSD 被加入集群后，已有OSD上部分PG将可能被挪到新OSD上；此时PG 和 OSD 的关系会发生改变。
*当已有的某 OSD down 了并变为 out 后，其上的 PG 会被挪到其它已有的 OSD 上。
*但是大部分的 PG 和 OSD 的关系将会保持不变，在状态变化时，Ceph 尽可能只挪动最少的数据。
*客户端根据 Cluster map 以及 CRUSH Ruleset 使用 CRUSH 算法查找出某个 PG 所在的 OSD 列表（其实是 up set）。

PG-Object-OSD的关系如下图所示：






## 概念

PG（放置组） — Placement Group，当数据到存储集群时，对象首先被映射至PG，然后再映射至OSDs。通过这样的方式我们可以将存储对象进行分组，进而减少对每个对象的元数据跟踪与使用时的开销（每个对象其放置历史的跟踪成本其实相当高）。增加PGs的数量可以减少每个OSD在集群中负载的方差（使OSDs负载更加平衡），但是越多的PG会消耗更多的CPU和内存。一般我们建议保持每100 PGs/OSD，虽然设置太高或太低时都会对并发性能，可靠性，数据平衡度产生影响，而这还将取决于群集的规模大小。

![](/assets/pg_1.png)

## 基本特点

* PG 确定了 pool 中的对象和 OSD 之间的映射关系。一个 object 只会存在于一个 PG 中，但是多个 object 可以在同一个 PG 内。
* Pool 的 PG 数目是创建 pool 时候指定的，Ceph 官方有推荐的计算方法。其值与 OSD 的总数的关系密切。当Ceph 集群扩展 OSD 增多时，根据需要，可以增加 pool 的 PG 数目。
* 对象的副本数目，也就是被拷贝的次数，是在创建 Pool 时指定的。该分数决定了每个 PG 会在几个 OSD 上保存对象。如果一个拷贝型 Pool 的size（拷贝份数）为 2，它会包含指定数目的 PG，每个 PG 使用两个 OSD，其中，第一个为主 OSD （primary），其它的为从 OSD （secondary），不同的 PG 可能会共享一个 OSD。
* Ceph 引入 PG 的目的主要是为了减少直接将对象映射到 OSD 的复杂度。
* PG 也是Ceph 集群做清理（scrubbing）的基本单位，也就是说数据清理是一个一个PG来做的。
* PG 和 OSD 之间的映射关系由 CRUSH 决定，而它做决定的依据是 CRUSH 规则（rules）。CRUSH 将所有的存储设备（OSD）组织成一个分层结构，该结构能区分故障域（failure domain），该结构中每个节点都是一个 CRUSH bucket。详细情况请阅读 CRUSH 相关的文档。
* PG 和 OSD 的关系是动态的
  * 一开始在 PG 被创建的时候，MON 根据 CRUSH 算法计算出 PG 所在的 OSD。这是它们之间的初始关系。
  * 部分 PG 和 OSD 的关系会随着 OSD 状态的变化而发生变化。
  * 当新的 OSD 被加入集群后，已有OSD上部分PG将可能被挪到新OSD上；此时PG 和 OSD 的关系会发生改变。
  * 当已有的某 OSD down 了并变为 out 后，其上的 PG 会被挪到其它已有的 OSD 上。
  * 但是大部分的 PG 和 OSD 的关系将会保持不变，在状态变化时，Ceph 尽可能只挪动最少的数据。
  * 客户端根据 Cluster map 以及 CRUSH Ruleset 使用 CRUSH 算法查找出某个 PG 所在的 OSD 列表（其实是 up set）。

PG-Object-OSD的关系如下图所示：

![](/assets/pg_2.png)

## 状态变化

* PG 的状态也是不断变化的，其主要状态包括：
  * Creating 创建中：PG 正在被创建。
  * Peering 互联中：表示一个过程，该过程中一个 PG 所在的所有 OSD 都需要互相通信来就 PG 的对象及其元数据的状态达成一致。处于该状态的PG不能响应IO请求。Peering的过程其实就是pg状态从初始状态然后到active+clean的变化过程。
  * Active 活动的：Peering 过程完成后，PG 的状态就是 active 的。此状态下，在主次OSD 上的PG 数据都是可用的。
  * Clean 洁净的：此状态下，主次 OSD 该 PG 中的对象，每没有异常，降级，或者需要重映射，所有副本都是就绪了。
  * Down：PG 所在的 OSD 掉线了，因为存放其某些关键数据（比如 pglog 和 pginfo，它们也是保存在OSD上）的 OSD down 了。
  * Degraded 降级的：某个 OSD 被发现停止服务 （down）了后，Ceph MON 将该 OSD 上的所有 PG 的状态设置为 degraded，此时该 OSD 的 peer OSD 会继续提供数据服务。这时会有两种结果：一是它会重新起来（比如重启机器时），需要再经过 peering 状态后，Ceph 会发起 recovery （恢复）过程，使该 OSD 上过期的数据被恢复到最新状态，最后到 clean 状态；二是 OSD 的 down 状态持续 300 秒后其状态被设置为 out，Ceph 会选择其它的 OSD 加入 acting set，并启动回填（backfilling）数据到新 OSD 的过程，使 PG 副本数恢复到规定的数目。
  * Recovering 恢复中：一个 OSD down 后，其上面的 PG 的内容的版本会比其它OSD上的 PG 副本的版本落后。在它重启之后（比如重启机器时），Ceph 会启动 recovery 过程来使其数据得到更新。
  * Backfilling 回填中：一个新 OSD 加入集群后，Ceph 会尝试级将部分其它 OSD 上的 PG 挪到该新 OSD 上，此过程被称为回填。与 recovery 相比，回填（backfill）是在零数据的情况下做全量拷贝，而恢复（recovery）是在已有数据的基础上做增量恢复。
  * Remapped 重映射：每当 PG 的 acting set 改变后，就会发生从旧  acting set 到新 acting set 的数据迁移。此过程结束前，旧 acting set 中的主 OSD 将继续提供服务。一旦该过程结束，Ceph 将使用新 acting set 中的主 OSD 来提供服务。
  * Stale 过期的：OSD 每隔 0.5 秒向 MON 报告其状态。如果因为任何原因，主 OSD 报告状态失败了，或者其它OSD已经报告其主 OSD down 了，Ceph MON 将会将它们的 PG 标记为 stale 状态。 

![](/assets/pg_3.png)

## 状态说明

|  |  |
| :--- | :--- |
|  |  |

状态    说明  
Creating    Ceph 仍在创建归置组  
Active    Ceph 可处理到归置组的请求  
Clean    Ceph 把归置组内的对象复制了规定次数  
Down    包含必备数据的副本挂了，所以归置组离线  
Replay    某 OSD 崩溃后，归置组在等待客户端重放操作  
Splitting    Ceph 正在把一归置组分割为多个  
Scrubbing    Ceph 正在检查归置组的一致性  
Degraded    归置组内的对象还没复制到规定次数  
Inconsistent    Ceph 检测到了归置组内一或多个副本间不一致（如各对象大小不一、恢复后对象还没复制到副本那里、等等）  
Peering    归置组正在互联  
Repair    Ceph 正在检查归置组、并试图修复发现的不一致（如果可能的话）  
Recovering    Ceph 正在迁移/同步对象及其副本  
Backfill    Ceph 正在扫描并同步整个归置组的内容，而不是根据日志推算哪些最新操作需要同步。 Backfill 是恢复的一种特殊情况  
Wait-backfill    归置组正在排队，等候回填  
Backfill-toofull    一回填操作在等待，因为目标 OSD 使用率超过了占满率  
Incomplete    Ceph 探测到某一归置组可能丢失了写入信息，或者没有健康的副本。如果你看到了这个状态，试着启动一下有可能包含所需信息的失败 OSD 、或者临时调整一下 min\_size 以完成恢复  
Stale    归置组处于一种未知状态——从归置组运行图变更起就没再收到它的更新  
Remapped    归置组被临时映射到了另外一组 OSD ，它们不是 CRUSH 算法指定的  
Undersized    此归置组的副本数小于配置的存储池副本水平  
Peered    此归置组已互联，但是不能向客户端提供服务，因为其副本数没达到本存储池的配置值（ min\_size 参数）。在此状态下可以进行恢复，所以此归置组最终能达到 min\_size


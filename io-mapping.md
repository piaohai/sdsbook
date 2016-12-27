Ceph 架构中客户端的读写操作均是以OSD上的RADOS对象存储中的数据对象（data object）为目标的，因此，Ceph的读写流程大致上需要走完 Pool-Object → Pool-PG → OSD-set → OSD-Disk 完整的链路，才能让客户端知道目标数据的object具体位置在哪里，具体来看：  


1. 创建资源池Pool以及相应的PG，根据上述的计算过程，PG在Pool被创建后就会被 MON根据CRUSH算法确认PG与对应的若干OSD的映射关系。也就是说，在客户端写入对象的之前，PG早已被创建完成，同时PG和OSD的映射关系也已经是确定了的。
2. Ceph客户端会通过哈希算法计算出存放数据对象（object）的PG的ID：
   1. 客户端输入pool ID和object ID（比如pool =“liverpool” and object-id =“john”）；
   2. ceph对object ID做哈希；
   3. ceph 对该 hash 值取 PG 总数的模，得到 PG 编号 （比如 58）（第2和第3步基本保证了一个 pool 中的所有 PG 将会被均匀地使用）;
   4. ceph 将  pool ID 和 PG ID 组合在一起（比如 4.58）得到 PG 的完整ID； 也就是：PG-id = pool-id.\(hash\(objet-id\) % PG-number\)
3. 客户端通过 CRUSH 算法计算出（或者说查找出） object 应该会被保存到 PG 中哪个 OSD 上。（注意：这里是说”应该“，而不是”将会“，这是因为 PG 和 OSD 之间的关系是已经确定了的，那客户端需要做的就是需要知道它所选中的这个 PG 到底将会在哪些 OSD 上创建对象。）。这步骤也叫做 CRUSH 查找。




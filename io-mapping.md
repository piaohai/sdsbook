Ceph 架构中客户端的读写操作均是以OSD上的RADOS对象存储中的数据对象（data object）为目标的，因此，Ceph的读写流程大致上需要走完 Pool-Object → Pool-PG → OSD-set → OSD-Disk 完整的链路，才能让客户端知道目标数据的object具体位置在哪里，具体来看：

1. 创建资源池Pool以及相应的PG，根据上述的计算过程，PG在Pool被创建后就会被 MON根据CRUSH算法确认PG与对应的若干OSD的映射关系。也就是说，在客户端写入对象的之前，PG早已被创建完成，同时PG和OSD的映射关系也已经是确定了的。

2. Ceph客户端会通过哈希算法计算出存放数据对象（object）的PG的ID：

   1. 客户端输入pool ID和object ID（比如pool =“liverpool” and object-id =“john”）；  
   2. ceph对object ID做哈希；  
   3. ceph 对该 hash 值取 PG 总数的模，得到 PG 编号 （比如 58）（第2和第3步基本保证了一个 pool 中的所有 PG 将会被均匀地使用）;  
   4. ceph 将  pool ID 和 PG ID 组合在一起（比如 4.58）得到 PG 的完整ID； 也就是：PG-id = pool-id.\(hash\(objet-id\) % PG-number\)

3. 客户端通过 CRUSH 算法计算出（或者说查找出） object 应该会被保存到 PG 中哪个 OSD 上。（注意：这里是说”应该“，而不是”将会“，这是因为 PG 和 OSD 之间的关系是已经确定了的，那客户端需要做的就是需要知道它所选中的这个 PG 到底将会在哪些 OSD 上创建对象。）。这步骤也叫做 CRUSH 查找。

   1. Ceph client 从 MON 获取最新的 cluster map；  
   2. Ceph client 根据上面的第（2）步计算出该 object 将要在的 PG 的 ID；  
   3. Ceph client 再根据 CRUSH 算法计算出 PG 中目标主和次 OSD 的 ID；

   也就是：OSD-ids = CURSH\(PG-id, cluster-map, crush-rules\)

4. 客户端写入数据

   在客户端使用 rbd 时一般有两种方法：

   1. 第一种 是 Kernel rbd。就是创建了rbd设备后，把rbd设备map到内核中，形成一个虚拟的块设备，这时这个块设备同其他通用块设备一样，一般的设备文件为/dev/rbd0，后续直接使用这个块设备文件就可以了，可以把 /dev/rbd0 格式化后 mount 到某个目录，也可以直接作为裸设备使用。这时对rbd设备的操作都通过kernel rbd操作方法进行的；
   2. 第二种是 librbd 方式。就是创建了rbd设备后，这时可以使用librbd、librados库进行访问管理块设备。这种方式不会map到内核，直接调用librbd提供的接口，可以实现对rbd设备的访问和管理，但是不会在客户端产生块设备文件；  

   应用写入rbd块设备的过程：

   1. 应用调用 librbd 接口或者对linux 内核虚拟块设备写入二进制块。下面以 librbd 为例；
   2. librbd 对二进制块进行分块，默认块大小为 4M，每一块都有名字，成为一个对象； 
   3. librbd 调用 librados 将对象写入 Ceph 集群； 
   4. librados 向主 OSD 写入分好块的二进制数据 \(先建立TCP/IP连接，然后发送消息给 OSD，OSD 接收后写入其磁盘\)；
   5. 主 OSD 负责同时向一个或者多个次 OSD 写入副本。注意这里是写到日志（Journal）就返回，因此，使用SSD作为Journal的话，可以提高响应速度，做到服务器端对客户端的快速同步返回写结果（commit）；
   6. 当主次OSD都写入完成后，主 OSD 向客户端返回写入成功； 

   也就是说，文件系统负责文件处理，librdb 负责块处理，librados 负责对象处理，OSD 负责将数据写入在Journal和磁盘中。

5. 下图为一个文件存放为例，说明了完整的计算过程  
   图为一个文件存放为例，说明了完整的计算过程：

   1. 一个 RBD image（比如虚机的一个镜像文件）会分成若干个 data objects 保存在 Ceph 对象存储中； 
   2. 一个 Ceph 集群含有多个 pool （使用 ceph osd pool create 命令创建pool）； 
   3. 一个 Pool 包含若干个 PG \(在创建 pool 时必须指定 pg\_num，在需要的时候对已有的pool的 pg\_num 也可以进行修改\)；
   4. 一个 PG 可以包含若干对象； 
   5. 一个 object 只在一个 PG 中； 
   6. 一个 PG 映射到一组 OSD，其中第一个 OSD 是主（primary），其余的是从（secondary）； 
   7. 许多 PG 可以映射到某个 OSD，通常一个OSD上会有100到200个PG； 

   几个比例关系：

   1. File ：object = 1 ： n （由客户端实时计算）； 
   2. object ：PG = n ： 1 （有客户端使用哈希算法计算）； 
   3. PG ：OSD = m ： n （由 MON 根据 CRUSH 算法计算）；




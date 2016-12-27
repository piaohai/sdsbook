## Monitor 高可用



## OSD高可用

OSD 的高可用是通过多方面共同保障的：

   * 冗余策略：通常每个 PG 的数据会以副本形式分布在多个 OSD 上，故一旦产生单点故障，该 PG 可以通过其他 OSD 继续对外提供服务
   * 心跳：OSD 出现单点故障后，需要有一定的机制来检测和判断单点故障的发生，并且能够做出相应的切换

在 CRUSH 一节 与 Pool 一节，我们已经介绍过了数据冗余策略，这里将不在展开。我们来看一下 OSD 是如何通过集群心跳实现高可用的，集群心跳分为三种：

   * OSD 与 OSD
   OSD进程与OSD进程相互之间会有心跳检查，称为OSD Ping，一个 OSD 会向它的所有同伴 OSD 发起心跳检测，就行网络 Ping 请求一样，来确认同伴 OSD 是否出现异常。心跳会同时对 public network 和 cluster network 两条网络进行检测，并且对两条网络检测的过程独立进行（关于 public network 与 cluster network 相关资料，请参见网络配置参考一节：[http://docs.ceph.org.cn/rados/configuration/network-config-ref/]("Ceph 网络配置参考")）。 两次心跳检查的通常为0.5 ～ 6.5s中间的一个随机数，每个OSD与OSD间进行心跳检测周期是不定的，这有助于以最快的速度发现异常的OSD，并且将其标记出集群。通常情况下，一个OSD向另一个OSD发起心跳检测后，发起方会注册心跳超时（默认为：20s），若超出20s后未收到对方心跳回应，则认为对方OSD有异常，并将自己认为有异常的OSD记录。

   * OSD 与 MON
OSD会定期的向Monitor报告状态，通常情况下为30s一次。但是如果该OSD此时通过心跳异常，记录下了异常的OSD，向Monitor汇报的周期会变短（默认为5s一次），这有助于让Monitor更及时的更新ClusterMap，保障集群的正常运行。Monitor会监测每个OSD定时向它汇报的信息，如果超出30s后一段时间，仍然没有任何该OSD的消息汇报，那么Monitor会将该OSD标记为Down。

MON Mark Down OSD
当Monitor收到某个OSD报告其他OSD状态异常时，Monitor不会轻信任何一个OSD的汇报，Monitor会记录有多少OSD同时报告该OSD异常，只有多个OSD同时报告某个OSD异常时，Monitor才能确信该OSD真的 Down掉，否则，当网络出现分区时，大量OSD将瞬间被标记为Down状态，导致整个集群无法正常工作。


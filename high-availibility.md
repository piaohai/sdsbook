## Monitor 高可用

## OSD高可用

OSD 的高可用是通过多方面共同保障的：

* 冗余策略：通常每个 PG 的数据会以副本形式分布在多个 OSD 上，故一旦产生单点故障，该 PG 可以通过其他 OSD 继续对外提供服务
* 心跳：OSD 出现单点故障后，需要有一定的机制来检测和判断单点故障的发生，并且能够做出相应的切换

在 CRUSH 一节 与 Pool 一节，我们已经介绍过了数据冗余策略，这里将不在展开。我们来看一下 OSD 是如何通过集群心跳实现高可用的，集群心跳分为三种：

* OSD 验证心跳  
  OSD进程与OSD进程相互之间会有心跳检查，称为OSD Ping，一个 OSD 会向它的所有同伴 OSD 发起心跳检测，就行网络 Ping 请求一样，来确认同伴 OSD 是否出现异常。心跳会同时对 public network 和 cluster network 两条网络进行检测，并且对两条网络检测的过程独立进行（关于 public network 与 cluster network 相关资料，请参见网络配置参考一节：[Ceph 网络配置参考](http://docs.ceph.org.cn/rados/configuration/network-config-ref/ "Ceph 网络配置参考")）。 两次心跳检查的间隔通常为0.5 ～ 6.5s中间的一个随机数，使得每个 OSD 与 OSD 间进行心跳检测周期是不确定的，这是为了花费最少的代价前提下，能以最快的速度发现异常的OSD，并且将其标记出集群。一个OSD向另一个OSD发起心跳检测后，发起方会注册心跳超时（默认为：20s），若超出20s未收到对方心跳回应，则认为对方 OSD 有异常，并将自己认为有异常的 OSD 记录下来。  
  ![](/assets/high_availibility_1.png)

* OSD 报告状态  
  OSD会定期的向 Monitor 报告自己的状态，如果一个 OSD 在 mon osd report timeout 时间（默认为 900s）内没向监视器报告，监视器就认为它 down 了。状态报告分为两类：

  * OSD 守护进程会向监视器报告某些事件，如某次操作失败、归置组状态变更、 up\_thru 变更、其他 OSD 是否死亡等，可以设置 \[osd\] 下的 osd mon report interval min 来更改最小报告间隔。

    ![](/assets/high_availibility_2.png)

  * OSD 守护进程每隔 120 秒会向监视器报告其状态，不论是否有值得报告的事件。在 \[osd\] 段下设置 osd mon report interval max 可更改OSD报告间隔。

    ![](/assets/high_availibility_3.png)

* OSD 标记死亡  
  在一个 OSD 被 Monitor 标记为死亡之前，至少有两个以上其他故障域中的 OSD 向 Monitor 报告该 OSD 死亡，该规则可以保证 Monitor 将多数 OSD 标记为死亡这种不合理的现象不会发生（当 OSD 与 OSD 之间通信不畅后，这些 OSD 会向 Monitor 互报对方 OSD 死亡，如果加以限制，Monitor 可能将多数 OSD 标记为死亡，而我们希望 Monitor 永远将少数的 OSD 标记为死亡）。用户可以通过配置参数 mon\_osd\_min\_down\_reporters 与 mon\_osd\_reporter\_subtree\_level 来改变默认的行为。

  ![](/assets/high_availibility_4.png)




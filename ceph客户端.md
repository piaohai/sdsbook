Ceph客户端通过集群交互协议 —— librados protocol，向外提供一种十分简单的访问方式。Client与Ceph 集群进行交互，只需要指定访问某个资源池（Pool）中的某个对象（Object），即可对集群中的Object进行操作，无需关心数据具体存放在什么位置。Ceph 客户端提供一下方式接入集群，实现不同业务场景下对数据的访问：

* 原生协议：Librados
* 非原生协议（只通过Librados库进行二次封装）：

  * 块设备访问方式 —— Librbd

  * 对象存储访问方式 —— RGW

  * 文件系统访问方式 —— Libcephfs

![](/assets/ceph_client_1.png)


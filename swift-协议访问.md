## 介绍 {#Swift概念介绍-介绍}

Swift是一个可以支持RESTful HTTP API，有着高稳定性并且可以管理大量无结构数据，低成本，提供多租户的对象存储服务。

## Swift的概念 {#Swift概念介绍-Swift的概念}

账户（account）：注意Swift的Account不一定是用户标示，可以理解存储区（storage area）。

容器（container）：容器有自己的metadata，包含了一组object。

对象（object）：包含具体数据和metadata。

集群（cluster）：表示一个Swift存储集群。

区域（region）：表示一个集群中物理隔离的部分。

区（zone）：表示物理隔离的节点，可用于控制故障域。

节点（node）：表示运行Swift进程的物理服务器。

## Swift的组件 {#Swift概念介绍-Swift的组件}

代理服务（swift-proxy-server）: 接受API请求或HTTP请求实现上传文件，修改元数据，创建容器等功能，也可以为浏览器提供文件或容器列表。为了提高性能，代理服务部署的时候有一个可选的memcache选项。

账户服务（swift-account-server）：管理对象存储定义的账户。

容器服务（swift-container-server）：管理对象存储内部的容器或文件夹映射。

对象服务（swift-object-server\):管理在存储节点上的真实存储对象（比如文件）。

注意Swift组件之间不需要通过rabbitmq进行通信。


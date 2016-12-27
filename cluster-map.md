Ceph 要求 Ceph 客户端和 OSD 守护进程需要知晓整个集群的拓扑结构，它们可以通过 Monitor 获取 cluster map 来达到这一点, Cluster Map 包括：

### Monitor Map 

### MON 集群的状态（包括 the cluster fsid， the position， name address and port of each monitor， 创建时间，最后的更新时间等）。例如：

```
1.    root@ceph1:/osd/data# ceph mon dump 
2.    dumped monmap epoch 1 
3.    epoch 1 
4.    fsid 4387471a-ae2b-47c4-b67e-9004860d0fd0 
5.    last_changed 0.000000 
6.    created 0.000000 
7.    0: 9.115.251.194: 6789/0 mon.ceph1 
8.    1: 9.115.251.195: 6789/0 mon.ceph2 
9.    2: 9.115.251.218: 6789/0 mon.ceph3
```

OSD Map  
当前所有 Pool 的状态和所有 OSD 的状态 （包括 the cluster fsid， map 创建和最后修改时间， pool 列表， replica sizes， PG numbers， a list of OSDs and their status \(e.g.， up， in\) 等）。通过运行 ceph osd dump 获取。例如：

```
1.    root@ceph1:~# ceph osd dump
2.    epoch 76
3.    fsid 4387471a-ae2b-47c4-b67e-9004860d0fd0
4.    created 2015-09-18 02:16:19.504735
5.    modified 2015-09-21 07:58:55.305221
6.    flags
7.    pool 0 'data' replicated size 3 min_size 2 crush_ruleset 0 object_hash rjenkins pg_num 64 pgp_num 64 last_change 1 flags hashpspool crash_replay_interval 45 stripe_width 0 
8.    osd.3 up in weight 1 up_from 26 up_thru 64 down_at 25 last_clean_interval [7，23) 9.115.251.194:6801/1204 9.115.251.194:6802/1204 9.115.251.194:6803/1204 9.115.251.194:6804/1204 exists，up d55567da-4e2a-40ca-b7c9-5a30240c895a 
9.    ......
```

PG Map  
包含PG 版本（version）、时间戳、最新的 OSD map epoch， full ratios， and 每个 PG 的详细信息比如 PG ID， Up Set， Acting Set， 状态 \(e.g.， active + clean\)， pool 的空间使用统计。可以使用命令 ceph pg dump 来获取 PG Map。  
CRUSH Map  
CRUSH （Controlled Replication under Scalable Hashing）Map：包含当前磁盘、服务器、机架等层级结构 （Contains a list of storage devices， the failure domain hierarchy \(e.g.， device， host， rack， row， room， etc.\)， and rules for traversing the hierarchy when storing data）。 要查看该 map 的话，先运行 ceph osd getcrushmap -o {filename} 命令，然后运行 crushtool -d {comp-crushmap-filename} -o {decomp-crushmap-filename} 命令，在vi 或者 cat {decomp-crushmap-filename} 即可。  
MDS Map  
MDS Map：包含当前所有 MDS 的状态 （包含当前 MDS 图的版本、创建时间、最近修改时间，还包含了存储元数据的存储池、元数据服务器列表、还有哪些元数据服务器是 up 且 in 的）。通过执行 ceph mds dump 获取，例如：

```
1.    root@ceph1:~# ceph mds dump
2.    dumped mdsmap epoch 13
3.    epoch   13
4.    flags   0
5.    created 2015-09-18 02:16:19.504427
6.    modified        2015-09-18 08:05:55.438558
7.    4171:   9.115.251.194:6800/962 'ceph1' mds.0.2 up:active seq 7
8.    5673:   9.115.251.195:6800/959 'ceph2' mds.-1.0 up:standby seq 1
```

MDS 只用于Ceph 文件系统该，与 RDB 和对象存储无关。


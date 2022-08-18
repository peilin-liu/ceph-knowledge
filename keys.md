# 重要知识点
## Ceph文件存储过程
 1. 一个文件将会切割为多个object（如1G文件每个object为4M将切割为256个），每个object会由一个由innode(ino)和object编号(ono)组成oid，即object id，oid是真个集群唯一，唯一标识object对象;
 2. 针对object id做hash并做取模运算，从而获取到pgid，即place group id，通过hash+mask获取到PGs ID，PG是存储object的容器;
 3. Ceph通过CRUSH算法，将pgid进行运算，找到当前集群最适合存储PG的OSD节点，如osd1和osd2（假设为2个副本）;
 4. PG的数据最终写入到OSD节点，完成数据的写入过程，当然这里会涉及到多副本，一份数据写多副本;
 5. 主OSD将数据同步给备份OSD，并等待备份OSD返回确认;
 6. 主OSD将写入完成返回给客户端。

## 块存储特性
 1. Thin-provisioned 瘦分配，即先分配特定存储大小，随着使用实际使用空间的增长而占用存储空间，避免空间占用；
 2. Images up to 16 exabytes 耽搁景象最大能支持16EB；
 3. Configurable striping 可配置的切片，默认是4M；
 4. In-memory caching 内存缓存；
 5. Snapshots 支持快照，将当时某个状态记录下载；
 6. Copy-on-write cloning Copy-on-write克隆复制功能，即制作某个镜像实现快速克隆，子镜像依赖于母镜像；
 7. Kernel driver support 内核驱动支持，即rbd内核模块；
 8. KVM/libvirt support 支持KVM/libvirt，实现与云平台如openstack，cloudstack集成的基础；
 9. Back-end for cloud solutions 云计算多后端解决方案，即为openstack，kubernetes提供后端存储；
 10. Incremental backup 增量备份；
 11. Disaster recovery (multisite asynchronous replication) 灾难恢复，通过多站点异步复制，实现数据镜像拷贝。

## 底层存储引擎bluestore与rocksdb
1. 由于leveldb依然需要磁盘文件系统支持，后去facebok对levelDB进行改进为RocksDB， RocksDB将对象数据的元数据保存在RocksDB，ceph对象数据放在了每个OSD中，那么就在当前OSD中划分出一部分空2间，格式化为BlueFS文件系统用于保存RocksDB中的元数据信息（成为Bluestore），并实现元数据的高可用，BlueStore最大的特点是构建在裸磁盘设备之上，并且对诸如SSD等新的存储设备做了很多优化工作。
3. 对全SSD及全NVMe SSD闪存适配。
4. 绕过本地文件系统层，直接管理裸设备，缩短IO路径。
5. 严格分离元数据和数据，提高索引效率。
6. 使用kv索引，解决文件系统目录结构遍历效率低的问题。
7. 支持多种设备类型。
8. 解决日志“双写”问题。
9. 期望带来至少2倍的写性能提升和同等读性能。
10. 增加数据校验及数据压缩等功能。

## Ceph CRUSH算法
1. Controllers replication under scalable hashing #可控的、可复制的、可伸缩的一致性hash算法。
2. ceph使用CRUSH算法来存放和管理数据，它是ceph的智能数据分发机制。ceph使用CRUSH算法来准确计算数据应该被保存到哪里，以及应该从哪里读取数据和保存元数据，不用的是CRUSH是按需计算出元数据，因此它就消除了对中心式的服务器/网关的需求，它使得ceph客户端能够计算出元数据，改过程也称为CRUSH查找，然后和OSD直接通信。
* 如果把对象直接映射到OSD之上会导致对象与OSD的对应关系过于紧密和耦合，当OSD由于故障发生变更时将会对整个ceph集群产生影响。
* 于是ceph将一个对象映射到RADOS集群的时候分为两步走。
  1. 首先使用一致性hash算法将对象名称映射到PG 2.7。
  2. 然后在PG ID基于CRUSH算法映射到OSD即可查找对象。
* 以上两个过程都是以实时计算的方式完成，而没有使用传统的查询数据与块设备的对应表的方式，这样有效避免了组件的中心化问题，也解决了查询性能和冗余问题。使得ceph集群扩展不在收查询的性能限制。
* 这个实时计算操作使用的就是CRUSH算法。
  1. Controllers replication under scalable hashing #可控的、可复制的、可伸缩的一致性hash算法。
  2. CRUSH是一种分布式算法，类似于一致性hash算法，用于为RADOS存储集群控制数据分配。

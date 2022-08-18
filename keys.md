# 重要知识点
## Ceph文件存储过程
### 1:一个文件将会切割为多个object（如1G文件每个object为4M将切割为256个），每个object会由一个由innode(ino)和object编号(ono)组成oid，即object id，oid是真个集群唯一，唯一标识object对象;
### 2:针对object id做hash并做取模运算，从而获取到pgid，即place group id，通过hash+mask获取到PGs ID，PG是存储object的容器;
### 3:Ceph通过CRUSH算法，将pgid进行运算，找到当前集群最适合存储PG的OSD节点，如osd1和osd2（假设为2个副本）;
### 4:PG的数据最终写入到OSD节点，完成数据的写入过程，当然这里会涉及到多副本，一份数据写多副本;
### 5:主OSD将数据同步给备份OSD，并等待备份OSD返回确认;
### 6:主OSD将写入完成返回给客户端。

## 块存储特性
### 1: Thin-provisioned 瘦分配，即先分配特定存储大小，随着使用实际使用空间的增长而占用存储空间，避免空间占用；
### 2:Images up to 16 exabytes 耽搁景象最大能支持16EB；
### 3:Configurable striping 可配置的切片，默认是4M；
### 4:In-memory caching 内存缓存；
### 5:Snapshots 支持快照，将当时某个状态记录下载；
### 6:Copy-on-write cloning Copy-on-write克隆复制功能，即制作某个镜像实现快速克隆，子镜像依赖于母镜像；
### 7:Kernel driver support 内核驱动支持，即rbd内核模块；
### 8:KVM/libvirt support 支持KVM/libvirt，实现与云平台如openstack，cloudstack集成的基础；
### 9:Back-end for cloud solutions 云计算多后端解决方案，即为openstack，kubernetes提供后端存储；
### 10:Incremental backup 增量备份；
### 11:Disaster recovery (multisite asynchronous replication) 灾难恢复，通过多站点异步复制，实现数据镜像拷贝。

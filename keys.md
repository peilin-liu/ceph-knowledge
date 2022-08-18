Ceph文件存储过程
1:一个文件将会切割为多个object（如1G文件每个object为4M将切割为256个），每个object会由一个由innode(ino)和object编号(ono)组成oid，即object id，oid是真个集群唯一，唯一标识object对象;
2:针对object id做hash并做取模运算，从而获取到pgid，即place group id，通过hash+mask获取到PGs ID，PG是存储object的容器;
3:Ceph通过CRUSH算法，将pgid进行运算，找到当前集群最适合存储PG的OSD节点，如osd1和osd2（假设为2个副本）;
4:PG的数据最终写入到OSD节点，完成数据的写入过程，当然这里会涉及到多副本，一份数据写多副本;
5:主OSD将数据同步给备份OSD，并等待备份OSD返回确认;
6:主OSD将写入完成返回给客户端。

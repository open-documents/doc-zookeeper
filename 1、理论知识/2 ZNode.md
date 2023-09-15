
Zookeeper树中的每个节点称为ZNode。

ZNode中维护了版本信息和时间戳。当数据发生变化时，版本号增加。

当客户端获取ZNode中的数据时，不仅会获取数据，还会获得数据对应的版本号。
当客户端更新或删除数据时，还必须一起提供所更新或删除数据的版本号。如果提供的更新或删除的版本号与ZNode中不匹配，那么更新或删除操作会失败（这个行为可以被重写）。

下面内容都根据JAVA API总结。

根据JAVA API中 CreateMode枚举类型的定义，创建的ZNode类型有以下7种：
- PERSISTENT（持久节点）：当客户端与Server失去连接时，创建的ZNode也不会被自动删除。
- EHPEMERAL（临时节点）：
- CONTAINER（容器节点）：
- SEQUENTIAL（顺序节点，个人认为不算单独的一种节点，只是一种创建节点的行为）：

ZNode中记录的统计信息（Stat）：
- czxid：创建该节点的zxid。
- ctime：创建该节点的时间，单位毫秒。 
- mzxid：最后修改该节点的zxid。
- mtime：最后修改该节点的时间，单位毫秒。
- pzxid：最后修改该节点子节点的zxid。
- version：该节点数据被修改的次数。
- cversion：该节点子节点被修改的次数。
- aversion：该节点ACL被修改的次数。
- dataLength：该节点数据的长度。
- numChildren：该节点子节点的数量。 
- ephemeralOwner：记录创建该临时节点的session id，如果该节点不是临时节点，返回0。 
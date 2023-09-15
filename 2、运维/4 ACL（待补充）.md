
Zookeeper使用ACL来进行对节点进行权限控制。特点：
- zookeeper的权限是基于znode节点的，需要对每个节点设置权限。
- znode节点支持同时设置多种权限方案和多个权限。当znode有多种权限的时候，只要有一个权限允许当前操作，即可执行当前操作。
- ACL只针对特定节点，而不会对其子节点也生效。
- 权限格式为 scheme：expression，permission。expression的格式是和scheme的种类有关的。

举个例子。如节点`/app` 的权限为 `ip:172.16.16.1,read`，子节点`/app/status`的权限为world readalbe，那么所有人都能访问/app/status。

# 授权策略（Scheme）

Zookeeper提供下面5种Scheme：

1）world：对于所有人进行赋予权限。在world授权模式下，id的取值只有一种，即为anyone。因此固定格式为 `world:anyone,permission`。



- ip：使用客户端IP作为ACL的id。对应的expression的格式为 addr/bits。
- digest：使用username:password字符串产生的MD5值作为ACL的id。
- auth：
- x509：

# Permission

Zookeeper支持下面的permission：
- CREATE：能够创建子节点。
- DELETE：能够删除子节点，仅限下一级。
- READ：能够获取节点的数据和显示子节点的列表。
- WRITE：能够设置节点的数据。
- ADMIN：能够设置或读取当前节点的权限列表。

ADMIN权限有特殊的作用：为了获取一个节点的ACL需要有READ或者ADMIN权限，但是没有ADMIN权限，用户的Hash值将会被隐藏。


# 插件式的ACL（AuthenticationProvider）

参考文档：https://zookeeper.apache.org/doc/r3.9.0/zookeeperProgrammers.html#ch_zkSessions（Pluggable ZooKeeper authentication部分）

没有理解，待补充

1）客户端创建一个句柄（handle）来连接到Zookeeper服务集群，句柄创建后的初始状态处于CONNECTING。

2）客户端尝试随机连接到Zookeeper集群中的一台服务器，Zookeeper服务端会创建一个64位表示的会话（Session），连接成功后，句柄状态转为CONNECTED。

正常情况下，客户端句柄在这两种状态中间转换。

3）如果发生了不能察觉的错误，如会话过期（expiration）、认证失败（authentication failure）、客户端显示地关闭会话，那么客户端句柄转为CLOSED状态。

# 会话的存活（alive）

会话的活性是通过客户端发送请求（可以是API提供的请求，也可以是PING请求）来保持的。

如果客户端和服务端之间的会话在一定时间没有通信，如客户端没有发送请求给服务端，那么客户端会发送一个PING请求给服务端。这个PING请求不仅能让服务端知道客户端的存活，还能让客户端知道与服务端的连接保持。

如果客户端在一定时间内没有接收到来自服务端的数据，那么客户端会认为连接丢失。此时
- 客户端的监听器（Watcher）会收到KeeperState类型为Disconnected的通知（关于KeeperState见...）
- 客户端的请求会抛出KeeperException.ConnectionLossException异常。（关于KeeperException见...）


# 创建会话的参数

See Also：
- 

## 1）Connection字符串

创建一个会话，必须提供一个连接字符串（逗号分隔的host:port字符串，每个片段代表一个集群中的主机。如"127.0.0.1:4545" 或 "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002"）。

客户端库尝试随机连接到其中的一台服务器。如果连接失败或者因为任何原因与已经连接的服务器断开连接，那么客户端会自动尝试连接Connection字符串中的下一台服务器，直到连接成功。

从3.2.0开始，Connection字符串后面能添加一个可选的"chroot"后缀。当调用客户端命令时，会将所有路径解析成相对"chroot"后缀的路径。例如，getXxx()、setXxx()、exists()等命令中的path，会被转换成 chroot/path。

## 2）超时时间（timeout）

创建一个会话，必须提供一个超时时间（timeout）。


## 3）监听器（Watcher） 

创建一个会话，必须提供一个默认监听器，这个监听器用来监听下面的事件：
- 客户端连接丢失（KeeperState.Disconnected）
- 会话过期（KeeperState.Expired）
- 连接建立（KeeperState.SyncConnected），此时事件类型为None，路径为null。

# Local Session

参考文档：https://zookeeper.apache.org/doc/r3.9.0/zookeeperProgrammers.html#ch_zkSessions（Local session部分）
Since：3.5.0



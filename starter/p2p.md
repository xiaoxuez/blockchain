## P2P



P2P网络结构可分为结构化和非结构化，非结构化组建方便，多个peer同时加入或下线离开，整个网络也会很稳定，结构化网络数据查询效率高，但当大量peer同时加入或离开时网络稳定性差，性能受影响。

结构化网络与纯分布式网络不同的是，所有节点按照某种结构有序组织，比如形成环状网络，或树状网络。DHT提出了一种网络模型，基于DHT，实现结构化网络的算法如Chord、Pastry、Kad等

DHT的核心为，资源空间和节点空间都进行编号，资源空间的编号可映射到对应节点空间。节点路由的普遍实现为trie树，形成K桶。资源的实现的话，邻近节点可存储冗余，对于热点信息，查询周围可缓存，Kad算法本身并未实现资源的冗余存储，主要是K桶的实现，逻辑的距离为节点Id的异或。

K桶一般为定时刷新，以自己为中心，查询距离自己最近的节点组成K桶，以太坊的Kad算法不太一样的是查询的是某个随机ID作为刷新基点，所查找的节点均在距离上向随机生成的TargetId收敛。



#### 示例

可查看[以太坊P2P解析](http://qjpcpu.github.io/blog/2018/01/30/shen-ru-ethereumyuan-ma-p2pmo-kuai-jie-dian-fa-xian-ji-zhi/)进一步理解。

[DHT的实现，BT种子嗅探器](https://github.com/shiyanhui/dht)



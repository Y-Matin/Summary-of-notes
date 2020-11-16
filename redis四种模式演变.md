### Redis的四种模式 单机、主从、哨兵、集群

#### 单机模式
> 只有一台机器部署了redis
- 优点： 部署简单、成本低、高性能，不需要同步数据。
- 缺点： 
    1. 可靠性不是很好，单节点有宕机的风险，
    2. 单机性能受限于cpu的处理能力。
> 使用场景：
   对并发性能要求不高，没有高可用需求，使用与简单的业务场景。

#### 主从复制
- 主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。

- 前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。

！[](https://yds-01.coding.net/p/Summary-of-notes/d/Summary-of-notes/git/raw/master/images/redis_master-slave.png)
##### 配置
```yml
1 slaveof <masterip> <masterport>
2 # 例如
3 # slaveof 192.168.1.214 6379
```
> 启动**主从**节点的所有服务，查看日志即可以看到**主从**节点之间的服务连接。

> 主节点负责数据的写入  
从节点负责数据的读取


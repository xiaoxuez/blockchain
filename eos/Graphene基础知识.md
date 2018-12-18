## Graphene 学习之基础知识

Graphene ，得益于 LMAX 架构和 DPoS 这一高效的共识机制, 使得基于 Graphene 成为目前最稳定和最高效的区块链底层

####  LMAX架构

**该架构主要基于：Disruptor + In Memory DDD + Event Sourcing**

1. 通过高并发框架（Disruptor）实现用户事件的输入和Domain Event的输出；
2. 一个常驻内存的Business Logic Processor（DDD领域模型），它负责在纯内存中处理业务逻辑；关键点：首先确保用户输入事件被持久化到数据库，并定时创建快照，然后在内存中响应事件更改业务对象的状态；因为一切都是在内存中处理，所以没有IO，也不需要数据库事务，非常快；
3. 机器down了怎么办？因为我们首先确保了业务对象的任何状态改变之前先持久化用户输入事件，所以在down机的时候通过事件回溯重新得到最新的业务对象。因为有了快照的保存，所以重建对象也非常快；

 

#### DPOS共识机制

...



希望后面学习完后能真正理解Graphene中的LMAX架构




# ADS

> 题型 多项选择题, 4主观题

- RPC socket API 比较

## consistency

- 出现的原因
- 一致性等级划分以及各个等级的区别以及对应的实现
- 解决方案具体实现, 可能考察具体细节(比如, redo中logging的变化情况等)

- 2PL如何实现sealizable 对example的判断
- multi-version解决了什么问题
- SI与sealizable的区别以及case判断

### 2PC
- 2PC为什么需要先voting
- 特殊情况例子  

### replicated state machine

### Paxos
- 只能达到两个条件，第三个条件的为什么达不到
- Paxos 代码修改之后case出错问题 协议设计的各个步骤, case

## Distributed System

### GFS
read write操作, 容错机制, 设计思想

### Chubby
特点，场景, 如何实现的Paxos, 与GFS的区别

## Distributed Computation

### data-parallel
- 前提, 架构, 实现
- interfance具体s
- 解决了什么问题，如何解决的
- 场景设计以及其设计理念

### Dryad
- channel 单机分布式区别
- 图计算以及其他计算的区别，为什么采用图计算
- 设计的合理性
- 主要框架

### graphlab
- 分析同步异步区别
- assumption

容错框架选择
为什么选择这个

## RPC

**socket API出现的问题**
> socket接口仅提供read/write接口; 程序员更加希望一个fuctional的接口; 为了让分布式计算对编程人员透明

**PRC实现**
> 通过pre-compilers(与编译器分离)创建node发送消息用以调用remote funcs; socket API是操作系统提供的接口, 其调用过程如下

<img src="rpc.png" title="RPC调用" weight="100" height="200"/>
> stub函数隐藏了网络细节(采用TCP进行网络传输)

** challenges **

+ *传指针* 由于pointer指向本机的位置, 所有的指针必须被解引用换成数值传入stub function
+ *传数值* 由于远端机器可能存在与本机不同的结构, 所以需要编码格式(JSON)

*LRPC* == lightweighted RPC
> 用于同一局域网内的相同机器之间的rpc调用, client stub代码会与kernel绑定并进行网络传输
> RPC 4次copy(stub -> RPC message message -> kernel kernel(client)->kernel(server) kernel(server) -> server's stub) LRPC 1次copy

[//]: <> (coherence指单个object的一致性, consistency指多个object的一致性)

## Consistency

[//]: <> (顺序一致性, 所有machine看到的执行结果是一样的; 释放一致性, 在对一个共享变量进行访问时, 进程在之前所有获得的锁毕竟完全释放)
[//]: <> (最终一致性, 即经过一段时间之后分布式server达成数据一致)
[//]: <> (因果一致性, 具有因果的operation[1.在同一个线程中先后执行 2.先读后写同一个数据 3.因果的传递性]具有一致的顺序)
一致性问题所遇到的挑战
> * ordering 每个server在update共享数值时产生update id: <time T, node ID>, 倘若产生冲突则server回滚所有update函数根据更新条目中的时间戳太确定重新执行顺序. 产生的问题: sync之后每个机器时间戳不同产生先后执行问题
> * 引入logical clock 如果先看到e1, 再执行e2 -> TS(e1) < TS(e2), 如何保持stable, 倘若TS > N 对于每个节点, 则有 N 之前的节点都是stable的; 可以通过master服务器管理commit seq No
> * COPS设计思想: 在写对象b时必须b的所有依赖(跟b具有因果关系)已经完全提交

## Crash Recovery & Logging

- All-or-nothing 要么all finish, 要么none of all, 即不存在中间状态
- straw man解决方案 对于写操作, 将整个page table写入disk(遇到的问题: 多个事务同时运行的时候可能将其他事务的中间状态写入disk)
- do-undo-redo协议: 将所有事务整合成一个log(append-only防止随机写; 通过链表维护Tx的先后顺序)
- logging协议: 先将log写入disk, 再将写操作写入disk, commit将一个commit节点放入log最后
- logging的恢复规则, 找到所有没有被CMT的log然后找到其祖先从最早的祖先的前一个节点的地方开始Redo
> (TODO: recovery rules如何redo的)
- REDO-UNDO log: 在crash之前已经CMT的就会被REDO(winner), 没有CMT的就会被UNDO(loser)，恢复时先UNDO之后再REDO
- 







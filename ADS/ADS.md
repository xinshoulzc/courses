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














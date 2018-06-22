## Flash 文件系统

- 与传统disk相比的优势以及缺点 价格差距逐年缩小 有限的读写次数 写操作之后需要擦除 读写速度、延迟远优于disk

### F2FS与LFS(log-structure system)
- 通过NAT(node adress table)减少了寻址时间
- wandering tree 在寻找文件位置时需要已经大量的节点
- multi-head logging 通过将log放置于不同的zone(hot, warm, cold)已提供多个流的读写
- 回收分散以及无效空间(通过SIT选择section, 通过SSA检查该块数据的正确性, 数据迁移, 标记为pre-free)
- 长时间使用后采用自适应logging(1. 将log记录在文件末尾, 可能需要清理 2. 重新利用dirty segment ???)
- 通过checkpoint进行断电后数据的恢复(仅仅保存主要信息)
- 回滚操作步骤(对N+n以及N-n的数据进行对比之后更新N+n的数据)

## 非易失性内存

- NVM特性 NVM/BPRAM 非易失性RAM PM 持久性内存 由于cache对内存进行写操作时存在reordering操作, 所以一般发生crash可能存在数据不一致的问题
- PMFS优缺点 轻量级以及与POSIX文件系统保持一致 内存映射的I/O 保护流浪写 pm_wbarrier引入Clflush强制保证缓存写入内存的一致性, Clflushopt保证高效地一致性读写, 通过pcommit实现一致性

###PMFS一致性保证
- copy on write 一般用于对文件数据的修改(拷贝, 修改数据, 指针专项转向)
- 日志 一般用于对元数据修改(追加型日志修改)
- 日志结构修改(LFS)
- PMFS采用混合方法(对于元数据采用细粒度logging, 即64比特的logging最小单元, 对于数据采用cow方法)
- redo undo(PMFS采用)区别, redo首先logging并写入tmp区域commit之后拷贝到fs, undo直接logging然后写入fs
- Atomic In-place Writes支持8, 16, 64位的原子写操作
- Mmap: Mmap in PMFS maps file data directly into the application’s virtual address space,出于性能的考虑。
- 解决reorder问题：将log entry的大小设置成与一个cacheline相同(64bytes)，并且保证对同一个cacheline的写操作不会被reordered。
- write protection: a). 利用SMAP,Prohibit writes into user area; b). PMFS write windows: mount as read-only, when writing CR0.WP is set to zero

## RAMCloud 
- server整体采用DRAM进行存储, 利用DRAM的低延迟(比flash快10x)同时保证大规模以及低延迟
- 数据模型采用KV-store（仅仅采用tableId, keyId, data就能读写）
- 不采用缓存技术(1. 1%的miss造成10%的性能下降 2. 不省钱)
- 采用disk进行数据备份(采用分块数据备份提高恢复数据的带宽), 采用日志方法提高IO读写以及避免高延迟
- Buffered logging(master服务器拥有所有的日志, R个备份服务器通过RPC调用备份logging的一个batch, 并且是异步调用, 备份服务器首先写到自己的DRAM, 然后写入本地的disk或者ssd)
- 采用log-structured的备份(保障性能以及disk的顺序读写), 通过哈希表对object进行读写
- (2-Phase Recovery) 数据恢复时候直接从R个服务器同时恢复数据(每个服务器只存储特定部分的数据, 只恢复数据位置, 不恢复数据内容, 通过user request恢复数据, 必须预留备份服务器)
- (Partitioned Recovery)将所有数据分块分散到所有master服务器云中, 一个master挂了之后数据可以备份到其他master中, 之后寻找一个新的备份服务器备份到此处
- 备份服务器挂了不恢复数据, 在运行过程中备份master数据

## lock (contended 竞争)

### 缓存一致性
- 由于多核设备在修改内存之后必须对其他核的缓存进行修改
- (directory-based 缓存一致性) 通过一个目录对缓存修改进行相应的记录, 读缓存之后会对相应的bit进行是否修改的check
- 多核竞争造成collapse的原因:
> 1) 缓存修改后其他所有核必须向修改核顺序读修改后的数据. 2) 修改数据的核仍然持有lock, 其他核需要获取lock之后才能进行相应的读操作 3) 大量lock竞争可能造成更长的等待时间
> 1) serial section比重越高, 片段越多collapse越快

### spin lock(back-off problem是什么???)
- 在ticket lock中，会需要记录两个变量，一个是now_serving表示正在使用lock的ticket（一个整数），另一个是next_ticket记录着当前最后一张ticket（就是拿票等待的核的号码）。当任何一个核去拿锁的时候，都会需要读取now_serving这个变量判断跟自己的ticket是否相等。这样一来，每个核都会对now_serving做cache，一旦这个锁被释放，ticket lock中的now_serving就会增加1，这个操作会invalidate所有核的cache里的now_serving，这会触发所有的核来重新读取now_serving这个值所在的cacheline，论文说明了在现有的架构中，这个read会被串行化处理，一个一个来，这就导致消耗的时间与等待锁的核的数量呈线性增长关系。

### MCS lock
- MCS是当一个正在使用锁的核把锁放掉之后，它会主动检查是不是有核在等这把锁，如果有会去通知这个核，代码上就是前一个核会去修改后一个核的is_locked变量，后一个核会一直spin在这个变量上，当这个变量被修改，它就知道自己能够拿到这把锁了。 所以，每个核都spin在自己的is_locked变量上，而不是全局去读某一个变量，这样就会有更好的scalability。

- non-scalable lock哪怕在N个核的时候表现不错，也很有可能在N+1或者N+2个核的时候突然collapse。scalable lock解决的就是这个问题，优点就是不会产生这种突然的大幅度的collapse，锁的性能不会随着核的数量增加而下降太大。缺点就是要对现有的源代码做改动，但是改动并不复杂。技术细节见第二个课后题。

## Transaction Memory

- 多核处理器越来越普及，如何利用硬件性能，更加高效的提升并行程序的性能成为难点。基于锁的同步机制存在以下问题：

> 粗粒度的锁虽然易于使用，但是性能低下，锁竞争频繁，无关的操作会出现顺序执行的情况
> 细粒度的锁虽然性能高，但是编程实现难度大，例如双端队列的并行版本使用细粒度的锁难以实现存在优先级反转（priority inversion）、护航（conveying）、死锁（dead lock）等问题
> 优先级反转一个高优先级任务间接被一个低优先级任务所抢先(preemtped)，使得两个任务的相对优先级被倒置
> 护航 多线程程序中相同优先级任务反复竞争同一个锁。当一个持有锁的进程被解调度了，可能是因为调度时间到了，发生了一个页错误，或者是其他一些终端。当有类似终端发生时，其他本该可以运行的进程却无法运行。

- 事务的属性: 原子性, 有序性
-  (read set) LT将共享内存中的值读出 (write set) LTX将共享内存中的值读出并更新 (write set) ST将值写入共享内存
- commit命令必须保证dataset未更新以及没有其他事务更新write set
- 具体执行流程 使用LT或者LTX指令从共享内存中读取数据 使用VALIDATE指令确认读取数据的有效性 使用ST指令来修改共享内存中的数据 使用COMMIT指令使修改生效；如果VALIDATE或者COMMIT失败，返回步骤1
- 缓存
为了最小化对非事务性内存指令影响，每个处理器持有两种cache：ƒ
regular cache （非事务性内存）
transactional cache (事务性内存)
这两种cache是互斥的，一次访问只可能访问一种cache，而且这两种cache都是一级cache或者二级cache，可以被处理器直接访问。regular cache是传统的直接映射cache；transactional cache空间较小，完全相连的由额外的逻辑来实现事务的提交和废弃。事务cache存储所有临时写的副本，除非事务提交，否则缓存内容将不会传给其他处理器或内存。

- 无锁的数据结构是一种能够无需使用锁就能支持多重读写操作的数据结构。使用无锁的数据结构我们能够避免大量因为维持全局锁操作所带来的额外开销。

- 孤儿事务 是指在被废弃（aborted）之后继续执行的事务（例如，在另一个已经提交过的事务更新了自己的read set之后）update: 孤儿事务是指本身事务在执行时产生了错误（而非读写数据被其他事务提前修改）导致的abort，以至于这个事务会一个retry占据资源无法commit/rollback.

- 事务在执行过程中利用VALIDATE指令来确保自己所读取的值是正确的，防止读取过期的数据，即保证了数据的一致性。

 - 事务性内存是基于事务具有短周期和小的数据集的假定。如果一个事务运行地越久，它就越有可能被中断或同步冲突所中止。同样地，数据集越大，其所需的事务性缓存也就越大，也就使得同步冲突发生的可能性越大。本实现过程不支持将冲突的事务提前处理，而是依赖于软件层面上的以加大冲突事务的间隔时间的方式来自适应的后移事务以减少事务的中止比例。本实验中的仿真所采用的缓存一致性协议提供了顺序一致性内存。但有一些研究者提出采用更加弱化的一致性协议以期达到更加高效的实现。如处理器一致性(Processor consistency)、弱一致性(week consistency)、释放一致性(release consistency)等等。

- 解决方案为了支持更大更长的事务，可以采用更加复杂且精心设计的硬件机制；为了降低事务中止的比例，可以采用硬件上的排队机制； 为了达到更加高效地实现，可以采用弱化的缓存一致性协议。如为了保障内存的顺序一致性，编程人员可以在每一个临界区的开始和结束时执行一个屏障指令(barrier instruction)。为提供事务性内存语义上的弱一致性内存，最直接的方式是使每一个事务性指令执行一个隐性障碍(implicit barrier)。


## NoSQL & BigTable
- Table、Tablet、SStable的关系. Tablet是从Table中若干范围的行组成的一个相当于原Table的子表，所以多个Tablet就能组成一个Tablet.而SSTable是Tablet在GFS文件系统中的持久化存储的形式，即我们平常说的Tablet，在Bigtable中，是存在一个一个SSTable格式的文件中的。 
- SSTable由许多block + index(表示了block的范围, 加快lookup速度)组成, Tablet与SSTable是多对多的关系
- 对一个table的读写操作: 写操作首先对GFS进行相应的commit log, 然后将生成相应的memtable; 读操作返回SStable/memtable reads/写操作之后的整合结果


- 这部分主要是针对Tablet服务器的，谷歌的Bigtable的读写请求都是通过Tablet服务器进行响应的。当一个写操作到达时，Tablet服务器首先要检查这个操作格式是否正确、操作发起者是否有执行这个操作的权限。权限验证的方法是通过从一个Chubby文件里读取出来的具有写权限的操作者列表来进行验证（这个文件几乎一定会存放在Chubby客户缓存里）。成功的修改操作会记录在提交日志里。可以采用批量提交方式来提高包含大量小的修改操作的应用程序的吞吐量。当一个写操作提交后，写的内容插入到memtable里面。当读操作到达时，Tablet服务器会作类似的完整性和权限检查。一个有效的读操作在一个由一系列SSTable和memtable合并的视图里执行，因为内存表和SSTable中都保存了数据。

- 由于BigTable针对数据存储进行包括压缩等各方面的优化，以及在事务的一致性上做出了让步，BigTable对于那些需要海量数据存储，高扩展性以及大量的数据处理，但又不要求强一致性的应用是十分适合的，比如Google Earth等。也因此，对于那些需要强一致性，需要同步更改多行数据的应用来说，BigTable是不合适的。

## NewSQL

- Spanner 采用无锁的分布式读事务 保证了外部一致性

### Spanner架构
- Universe：Spanner的整个部署 universe master：单例，维护所有zones； placement driver：单例，负责在zones之间迁移数据。 zone：等同于其下BigTable的部署节点，是管理配置的单位，也是物理隔离的单位；
- Zone内部： zonemaster：每个zone都有一个，负责将数据分配给当前zone的spanserver； location proxy：每个zone有多个，为client提供定位到需要的SpanServer的服务； spanserver：每个zone有成百上千个，负责为client提供数据服务；

### 外部一致性
EC.强调的是，每个txn在系统中生效的时间点和他们的commit timestamp保持一致，即对于commit timestamp t1 < t2，则所有相关server看到的生效顺序也会是先T1后T2；而Serialization强调的是，txn之间要有执行的先后顺序，但不能保证在分布式里所有server上看到的顺序都是一致的（可能会在某server上看到的生效顺序和每个txn设定的全局顺序不一致。限制松一点）。

- How does Spanner achieve the external consistency? 首先，Spanner通过巧妙的设计与同步得到TrueTime API，调用时能够获得一个较精确的interval，能保证绝对时间在这个范围内；然后，所有需要全局同步的txn需要进行两步提交（2PC: prepare-commit）：在prepare阶段调用TrueTime.now()，将这个txn的commit timestamp（记为s）设置为TrueTime.now().latest，从而保证s晚于当前绝对时间之前的所有txn提交时间；然后等待一段时间直到TrueTime.after(s)为true，即此时所有先于s提交的txn都全局同步完成了，再进行commit阶段提交这个txn。这样，保证了当前txn提交的timestamp一定大于之前的所有txn，且之前的所有txn都全局可见。

- TT发生错误会怎么办. 可能导致txn取得的commit timestamp会小于绝对时间在这之前提交的txn，使得实际发生在之后的txn可见时，之前的txn可能不可见。违反external consistency。“经过测试，我们的TrueTime足够精确，出错的概率非常小。”

## Transaction chopping

### 分片过程
- 在不同事务执行过程中途释放锁使得其他事务能够执行
- 事务由小片段(pieces), 其中每小片是原子执行的，小片之间通过S-edges连接，潜在冲突的pieces(两个片段访问同一个table并且至少有一个是write)之间通过C-edges连接
- 将SC-cycle合并成一个单独的片段

### transaction chains (???)

## RDMA
- HTM(hardware transaction memory) RDMA(Remote Direct Memory Access)
- HTM: 在硬件层面提供了 原子性 支持(ACI, 注意没有D)限制 1)HTM内不能有IO 2) HTM不保证一定成功, 需要fallback
- RDMA: 提供了 强一致性 的远程访问支持 注意, 这里的一致性是内存层面的一致性, 当作Coherence来理解 强一致性即: read last write 对外提供三种接口 1)模拟TCP/IP: ~100us 2) recv/send: ~7us 3)单向访问: 最快, ~3us级(原生TCP/IP大约是500us)RDMA CAS有多种级别. 在论文所有的机器上, RDMA CAS与Local CAS 不兼容

### 读写事务
+ 思路:
    + 对于read_set与write_set, 判断是否在都在本地, 如果不是, 则有RDMA加锁并读取(加锁之后, 其他事务就无法访问它们了)
        + end-time = minimum(endtime of lock(x) for x in read_set): 取最小的lease time
    + 涉及到的数据都到了本地后, 使用HTM进行事务处理
    + 事务提交前, 检查now < end-time - DELTA, 如果为真, 说明所有的读锁都没有过期, 事务可以提交, 否则abort
    + 事务提交后, 将write_set加的数据写回, 并释放写锁, 读锁会自动过期
+ 读写事务冲突检测:
    + 先本地/再本地: 由HTM保证原子性
    + 先远程/再本地: 远程的RDMA会先加锁数据, 本地HTM要去访问时, 如果发现数据上有写锁, 则abort, 如果有读锁, 则可并行执行
    + 先远程/再远程: 远程的RDMA会先加锁数据, 另一个远程RDMA来访问时, 如果发现已经有写锁, 则中止, 如果发现是读锁, 则取其中止时间为自己锁的中止时间, 然后正常执行
+ Fallback实现
    + HTM不保证一定可以成功, 即使没有冲突, 因此, 需要一个fallback(基于锁)
    + 步骤 放锁 对所有锁排序后重新加锁, 以避免死锁 执行操作 放锁 写回
    + 注意: 以上操作需要记Log, 以应对错误情况

### 重点强调
+ 冲突检测
    + 对于同样使用了HTM的两个事务, HTM保证它们的before-or-after
    + 对于一个已经存在的HTM事务(读ab, 写cd), 如果另一个远程事务需要访问abcd, 则需要先用RDMA加锁, 并把数据读回来, 加锁过程会abort已有的HTM事务, 从而后面的这个远程事务会先发生. 之前的HTM事务需要重试
    + 对于两个远程事务, 锁会保证其先后
    + 所有只读事务都当作远程事务来处理(使用加锁来保证顺序)
+ HTM + 读写锁机制等价于2PL, 从而, 可以保证Serializability
    + 使用HTM完成的事务与使用2PL完成的事务是等价的. 
        + 在HTM中, 如果两个事务冲突, 且其中至少一个为写, 则HTM保证abort掉其中一个, 从而保证了顺序.
        + 在HTM中, 如果两个事务冲突, 且其中一个涉及到了远程对象, 且加锁机制可以保证两个事务的顺序
    + DrTM中的基于Lease的读锁是于传统的shared read lock是等价的
        + 在DrTM中, 如果机器A对x加了读锁, 则机器B可以共享此读锁. 
        + 当事务执行完毕, DrTM会检测所有读锁是否过期, 如果已经过期, 事务不会commit, 没有影响. 如果没有过期, 则提交. 这等价于2PL中, 所以锁在Shrink阶段提交, 并且不会加新的锁.
    + DrTM中, 所以锁都会在`Shrink`阶段提交, 等价于2PL中的行为
        + HTM执行完毕后, 所有本地'锁'都会被释放.
        + 在HTM的confirmation阶段, 即检测读锁是否过期, 检测通过, 则先行于所有锁释放, 且不会再拿新的锁.
        + 在所有HTM事务提交后, 写锁最终都被会释放.

### 存储层
+ DrTM提供了两类存储 有序存储: 基于B+ Tree, 没有充分利用RDMA的特性 无序存储: 基于Hash, 充分利用了RDMA的特性, 并且, 在事务层的保护下, 不需要考虑冲突检测问题

+ DrTM的优点大概是 分离了存储操作与冲突检测 之前的实现偏向于RDMA(而不是Local), 增加了延迟 本地内存操作还是比RDMA快, 但是之前的实现没有考虑本地缓存

### 缺点
+ 事务需要事先知道write-set与read-set
+ 无序存储是HTM/RDMA友好的, 而有序存储则没有对RDMA做过优化
+ 当有机器失效时, 考虑Durability而不是可用性
    + 读写事务提交后, 需要写回, 并释放写锁, 如果此时, 此机器挂了, 其锁永远无法被释放

### 课后习题
+ How does DrTM detect conflicts between remote reads and local writes? Local writes会被包裹在HTM中. 远程reads使用的RDMA会打断本地HTM, 从而避免了冲突
+ Why DrTM has the deadlock issue and how does it avoid the deadlock? 在fallback handler中会出现, 详见上文事务层中fallback实现




## Graph Query on RDF

- RDF瓶颈 一共有两种不同的实现，分别是 Triple store and triple join 和 Graph store and graph exploration。前者是以 triple 的方式来将 RDF 数据存储在关系型数据库中，因此查询有两个步骤，scan 和 join。scan 会分为子查询，最后再借由 hash join 之类的 join 的操作将查询的结果 join 在一起。由此可知如果数据非常大的时候，最后的 join 会是很大的问题。
第二种方式是以图的方式来存储和查询 RDF。这样的方式以 Trinity.RDF 为代表，有一些剪枝的优化。但是最后也会有一个 final join 的过程。

- 悟空和原先设计的不同以及优势最大的不同在于索引的存储方式。之前的基于图的设计都是用独立的索引数据结构来存索引，但是 Wukong 是把索引同样当做基本的数据结构（点和边）来存储。并且会考虑分区来存储这些索引。
这样做有两个好处，第一点就是在进行图上的遍历或者搜索的时候可以直接从索引的节点开始，不用做额外的操作。第二点是这样使得索引的分布式存储变得非常简单，复用了正常的数据的存储方式。

- full-history剪枝与先前技术有何不同, Wukong为何可以这么做Full-history 就是说所有的历史记录都会被记录下来。之前是只记录一次的。之所以可以这样做是因为一方面 RDF 的查询都不会有太多步，而且 RDMA 在低于 2K bytes 的时候性能都是差不多的，所以 Wukong 可以这样做。

## latency

- 出现latency的原因 1)资源竞争 2)倾斜的访问模式 3)排队延迟 4)后台活动
- 降低组件变量延迟 1)区分服务类型和高级队列 2)减少队首阻塞 3)管理后台活动

### 容延技巧（Tail-tolerant techniques）
- Within request short-term adaptations Hedged requests(对冲请求) 把同样的request发布给多台有data replica的servers，哪个server 响应最快，就使用这个响应结果。 Tied request 一旦有服务器发生相应则通知所有执行该request执行的服务器
- Cross request long-term adaptations Micro-partitions 产生远多于现存机器数的细小的分区，进行动态分配partitions，保证这些机器上的负载均衡 Selective Replication 预测可能的负载不均衡，创造额外的复制品 Latency-induced Probation 观察不同机器，暂时排除特别慢的机器，对excluded servers继续发送shadow请求，一旦发现问题减缓， 把这些排除掉的机器再合并进来。
- Trends that further hurt the latency 1) 硬件层面的差距 2) 设备异构 3) 系统规模的上升
- Trends that help mitigate the latency 1) 提高带宽 2) 减少额外开销

## NFV

- 为什么要提出这个技术？与传统的区别？现有的大多数路由器的设计都是封闭的，静态的和刻板的。 网络管理员很难指定甚至定义不同functions之间的交互。 而且，要实现新的functions很难。 与传统的区别：（1）灵活:很容易添加新的功能 （2），模块化：通过组合元素来实现功能； （3）， Open：允许开发者添加元素（Elements);（4），效率：与硬件性能相差不大。

- Pull和Push push和pull是Click里elements直接的两种连接方式。 Push是从源Element到下游的元素，通过事件触发，比如有包到达。在Push连接方式里，上游的元素会提交一个包到下游的元素； Pull是从目的地元素到上游元素。在Pull的连接方式里，是下游的元素发送包请求到上游的元素。当输出接口准备好发包时，会开始一个pull过程：该过程会一直向后遍历图直到遇到某个元素‘吐出’一个包。Push 和 Pull的共同使用可以使包转发连接的适当终止，可以很好地解决路由器控制流问题。

- Click is the difficulty of scheduling CPU time among pull and push paths Click 的调度器就是一个 Pull Element，它有多个输出，而只有一个输入。至于 CPU 时间调度的问题， 是说 Click 没办法处理多个设备同时接收或者发送数据的情况。目前的处理方法是 linux 处理大部分这样的调度，剩下的交由 Click 来做。最终所有这些都应该由一个单一的机制来控制。关于 improve 可以把 linux 里相关的逻辑作为一个 Element 引入，不知是否可行。

- Dropping Policies Click通过队列元素来实现一个简单的Dropping策略， 也就是当包数目超过配置的最大长度时，这些包都会被扔掉。 论文提到的Dropping策略：（1），RED：Random Early Detection。 该Element以下游的最近的队列长度作为Dropping的依据。(2), RED over multiple queues: 如果RED的下游有多个队列，那就将这些队列的长度都加起来作为Dropping的依据；（3），weight RED: 每个包根据它的优先级有不同的Drop的概率。

- NFV:Network Function Virtualization Network Functions(Middleboxes): Firewall, IDS, DNS。网络功能虚拟化就是虚拟化这些网络功能，也就是通过软件模拟实现这些网络功能。 或者说是：运行在云设施的网络服务。 NFV目标：（1）省钱：使用更便宜的商业服务器来实现网络功能；减少专有硬件的使用，从而降低能耗，并且能够方便地维护。（2）赚钱：加速网络服务部署； 网络基础设施作为服务；









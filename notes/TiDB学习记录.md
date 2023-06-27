# TiDB学习记录

## 简介

TiDB 是 PingCAP 公司自主设计、研发的**开源分布式关系型数据库**，是一款同时支持在线事务处理与在线分析处理 (Hybrid Transactional and Analytical Processing, **HTAP**) 的融合型分布式数据库产品，具备水平扩容或者缩容、金融级高可用、实时 HTAP、云原生的分布式数据库、兼容 MySQL 5.7 协议和 MySQL 生态等重要特性。目标是为用户提供一站式 OLTP (Online Transactional Processing)、OLAP (Online Analytical Processing)、HTAP 解决方案。TiDB 适合高可用、强一致要求较高、数据规模较大等各种应用场景。

### 关键技术创新

#### 数据架构设计目标

- 扩展性
- 强一致性（副本一致性），高可用性
- 标准SQL，支持ACID事务
- 云原生（弹性）
- HTAP
- 兼容主流生态与协议

#### 基础数据技术元素

##### 数据模型

- 关系模型
- 文件模型
- 对象模型
- Key-Value模型

##### 数据存储与检索结构

- B树
- LSM树
- 分形树

##### 数据格式

- 结构化数据
- 半结构化数据
- 非结构化数据

##### 存储引擎（负责数据的存取和持久化）

- InnoDB
- RocksDB

##### 复制协议

- Raft
- Paxos

##### 分布式事务模型

- XA
- Percolator

##### 数据架构

- share-disk
- share-nothing
- share-everything

##### 优化器算法

根据数据分布和统计信息生成成本最低的执行计划

##### 执行引擎

- 火山引擎
- 向量化
- 大规模并行计算

##### 计算引擎

### 五大核心特性

- 一键水平扩容或者缩容
  - 按需对计算、存储分别进行在线扩容或者缩容，扩容或者缩容过程中对应用运维人员透明。
- 金融级高可用
  - 数据采用多副本存储，数据副本通过 Multi-Raft 协议同步事务日志，多数派写入成功事务才能提交，确保数据强一致性且少数副本发生故障时不影响数据的可用性。
  - 可按需配置副本地理位置、副本数量等策略满足不同容灾级别的要求。
- 实时 HTAP
  - 提供行存储引擎 TiKV、列存储引擎 TiFlash 两款存储引擎，TiFlash 通过 Multi-Raft Learner 协议实时从 TiKV 复制数据，确保行存储引擎 TiKV 和列存储引擎 TiFlash 之间的数据强一致。
  - TiKV、TiFlash 可按需部署在不同的机器，解决 HTAP 资源隔离的问题。
- 云原生的分布式数据库
- 兼容 MySQL 5.7 协议和 MySQL 生态

## 整体架构

### 三大核心组件

#### TiDB Sever

- 处理客户端的连接
- SQL语句的解析和编译
- 关系型数据库与KV的转化
- SQL语句的执行
- 在线DDL的执行
- 垃圾回收

#### Storage cluster

##### TiKV

- 数据持久化
- 分布式事务支持
- 副本的强一致性和高可用性
- MVCC
- Coprocessor（算子下推）

##### TiFlash

- 列式存储提高分析查询效率
- 支持强一致性和实时性
- 业务隔离
- 智能选择

#### PD(Placement Driver) cluster

- 整个集群TiKV的元数据存储
- 分配全局ID和事务ID
- 生成全局时间戳TSO
- 收集集群信息进行调度
- 提供TiDB Dashboard服务

## 存储

### Key-Value Pairs

TiKV 数据存储的两个关键点：

- 这是一个巨大的 Map（可以类比一下 C++ 的 std::map），也就是存储的是 Key-Value Pairs（键值对）

- 这个 Map 中的 Key-Value pair 按照 Key 的二进制顺序有序，也就是可以 Seek 到某一个 Key 的位置，然后不断地调用 Next 方法以递增的顺序获取比这个 Key 大的 Key-Value。

### 本地存储 (RocksDB)

TiKV 没有选择直接向磁盘上写数据，而是把数据保存在 RocksDB 中，具体的数据落地由 RocksDB 负责（这里可以简单的认为 RocksDB 是一个单机的持久化 Key-Value Map）。

这个选择的原因是开发一个单机存储引擎工作量很大，特别是要做一个高性能的单机引擎，需要做各种细致的优化，而 RocksDB 是由 Facebook 开源的一个非常优秀的单机 KV 存储引擎，可以满足 TiKV 对单机引擎的各种要求。

### Raft 协议

Raft 是一个一致性协议，提供几个重要的功能：

- Leader（主副本）选举
- 成员变更（如添加副本、删除副本、转移 Leader 等操作）
- 日志复制

TiKV 利用 Raft 来做数据复制，每个数据变更都会落地为一条 Raft 日志，通过 Raft 的日志复制功能，将数据安全可靠地同步到复制组的每一个节点中。

不过在实际写入中，根据 Raft 的协议，只需要同步复制到多数节点，即可安全地认为数据写入成功。

数据的写入是通过 Raft 这一层的接口写入，而不是直接写 RocksDB。

### Region

为了实现存储的水平扩展，数据将被分散在多台机器上。对于一个 KV 系统，将数据分散在多台机器上有两种比较典型的方案：

- Hash：按照 Key 做 Hash，根据 Hash 值选择对应的存储节点。
- Range：按照 Key 分 Range，某一段连续的 Key 都保存在一个存储节点上。

TiKV 选择了第二种方式，将整个 Key-Value 空间分成很多段，每一段是一系列连续的 Key，将每一段叫做一个 **Region**，并且会尽量保持每个 Region 中保存的数据不超过一定的大小，目前在 TiKV 中默认是 96MB。每一个 Region 都可以用 **[StartKey，EndKey)** 这样一个左闭右开区间来描述。

将数据划分成 Region 后，TiKV 将会做两件重要的事情：

- 以 Region 为单位，将数据分散在集群中所有的节点上（由PD负责），并且尽量保证每个节点上服务的 Region 数量差不多（Region 的副本数与 TiKV 实例数量无关）。
- 以 Region 为单位做 Raft 的复制和成员管理。TiKV 是以 Region 为单位做数据的复制，也就是一个 Region 的数据会保存多个副本，TiKV 将每一个副本叫做一个 Replica。Replica 之间是通过 Raft 来保持数据的一致，一个 Region 的多个 Replica 会保存在不同的节点 上，构成一个 Raft Group。其中一个 Replica 会作为这个 Group 的 Leader，其他的 Replica 作为 Follower。默认情况下，所有的读和写都是通过 Leader 进行，读操作在 Leader 上即可完成，而写操作再由 Leader 复制给 Follower。

### MVCC

TiKV 的 MVCC 实现是通过在 Key 后面添加版本号来实现，简单来说，没有 MVCC 之前，可以把 TiKV 看做这样的：

```
Key1 -> Value
Key2 -> Value
……
KeyN -> Value
```

有了 MVCC 之后，TiKV 的 Key 排列是这样的：

```
Key1_Version3 -> Value
Key1_Version2 -> Value
Key1_Version1 -> Value
……
Key2_Version4 -> Value
Key2_Version3 -> Value
Key2_Version2 -> Value
Key2_Version1 -> Value
……
KeyN_Version2 -> Value
KeyN_Version1 -> Value
……
```

对于同一个 Key 的多个版本，版本号较大的会被放在前面，版本号小的会被放在后面（见 Key-Value 一节，Key 是有序的排列），这样当用户通过一个 Key + Version 来获取 Value 的时候，可以通过 Key 和 Version 构造出 MVCC 的 Key，也就是 Key_Version。然后可以直接通过 RocksDB 的 SeekPrefix(Key_Version) API，定位到第一个大于等于这个 Key_Version 的位置。

### 分布式 ACID 事务

TiKV 的事务采用的是 Google 在 BigTable 中使用的事务模型：Percolator ，TiKV 根据这篇论文实现，并做了大量的优化。

## 计算

### 表数据与 Key-Value 的映射关系

- 为了保证同一个表的数据放在一起，方便查找，TiDB 会为每个表分配一个表 ID，用 `TableID` 表示。表 ID 是一个整数，在整个集群内唯一。
- TiDB 会为表中每行数据分配一个行 ID，用 `RowID` 表示。行 ID 也是一个整数，在表内唯一。对于行 ID，TiDB 做了一个小优化，如果某个表有整数型的主键，TiDB 会使用主键的值当做这一行数据的行 ID。

每行数据按照如下规则编码成 (Key, Value) 键值对：

```avrasm
Key:   tablePrefix{TableID}_recordPrefixSep{RowID}
Value: [col1, col2, col3, col4]
```

### 索引数据和 Key-Value 的映射关系

TiDB 同时支持主键和二级索引（包括唯一索引和非唯一索引）。与表数据映射方案类似，TiDB 为表中每个索引分配了一个索引 ID，用 `IndexID` 表示。

对于主键和唯一索引，需要根据键值快速定位到对应的 RowID，因此，按照如下规则编码成 (Key, Value) 键值对：

```asciidoc
Key:   tablePrefix{tableID}_indexPrefixSep{indexID}_indexedColumnsValue
Value: RowID
```

对于不需要满足唯一性约束的普通二级索引，一个键值可能对应多行，需要根据键值范围查询对应的 RowID。因此，按照如下规则编码成 (Key, Value) 键值对：

```dust
Key:   tablePrefix{TableID}_indexPrefixSep{IndexID}_indexedColumnsValue_{RowID}
Value: null
```

上述所有编码规则中的 `tablePrefix`、`recordPrefixSep` 和 `indexPrefixSep` 都是字符串常量，用于在 Key 空间内区分其他数据，定义如下：

```ini
tablePrefix     = []byte{'t'}
recordPrefixSep = []byte{'r'}
indexPrefixSep  = []byte{'i'}
```

### 元信息管理

TiDB 中每个 `Database` 和 `Table` 都有元信息，也就是其定义以及各项属性。这些信息也需要持久化，TiDB 将这些信息也存储在了 TiKV 中。

每个 `Database`/`Table` 都被分配了一个唯一的 ID，这个 ID 作为唯一标识，并且在编码为 Key-Value 时，这个 ID 都会编码到 Key 中，再加上 `m_` 前缀。这样可以构造出一个 Key，Value 中存储的是序列化后的元信息。

### SQL 层简介

TiDB 的 SQL 层，即 TiDB Server，负责将 SQL 翻译成 Key-Value 操作，将其转发给共用的分布式 Key-Value 存储层 TiKV，然后组装 TiKV 返回的结果，最终将查询结果返回给客户端。

这一层的节点都是无状态的，节点本身并不存储数据，节点之间完全对等。

#### SQL 运算

最简单的方案就是通过上一节所述的表数据与 Key-Value 的映射关系方案，将 SQL 查询映射为对 KV 的查询，再通过 KV 接口获取对应的数据，最后执行各种计算。

#### 分布式 SQL 运算

计算应该需要尽量靠近存储节点，以避免大量的 RPC 调用。

#### SQL 层架构

用户的 SQL 请求会直接或者通过 `Load Balancer` 发送到 TiDB Server，TiDB Server 会解析 `MySQL Protocol Packet`，获取请求内容，对 SQL 进行语法解析和语义分析，制定和优化查询计划，执行查询计划并获取和处理数据。数据全部存储在 TiKV 集群中，所以在这个过程中 TiDB Server 需要和 TiKV 交互，获取数据。最后 TiDB Server 需要将查询结果返回给用户。

## 调度

PD (Placement Driver) 是 TiDB 集群的管理模块，同时也负责集群数据的实时调度。

### 调度的需求

**第一类：作为一个分布式高可用存储系统，必须满足的需求，包括几种**

- 副本数量不能多也不能少
- 副本需要根据拓扑结构分布在不同属性的机器上
- 节点宕机或异常能够自动合理快速地进行容灾

**第二类：作为一个良好的分布式系统，需要考虑的地方包括**

- 维持整个集群的 Leader 分布均匀
- 维持每个节点的储存容量均匀
- 维持访问热点分布均匀
- 控制负载均衡的速度，避免影响在线服务
- 管理节点状态，包括手动上线/下线节点

满足第一类需求后，整个系统将具备强大的容灾功能。满足第二类需求后，可以使得系统整体的资源利用率更高且合理，具备良好的扩展性。

### 调度的基本操作

调度的基本操作指的是为了满足调度的策略。上述调度需求可整理为以下三个操作：

- 增加一个副本
- 删除一个副本
- 将 Leader 角色在一个 Raft Group 的不同副本之间 transfer（迁移）

刚好 Raft 协议通过 `AddReplica`、`RemoveReplica`、`TransferLeader` 这三个命令，可以支撑上述三种基本操作。

### 信息收集

调度依赖于整个集群信息的收集，简单来说，调度需要知道每个 TiKV 节点的状态以及每个 Region 的状态。TiKV 集群会向 PD 汇报两类消息，TiKV 节点信息和 Region 信息：

#### 每个 TiKV 节点会定期向 PD 汇报节点的状态信息

TiKV 节点 (Store) 与 PD 之间存在心跳包，一方面 PD 通过心跳包检测每个 Store 是否存活，以及是否有新加入的 Store；另一方面，心跳包中也会携带这个 Store 的状态信息。

#### 每个 Raft Group 的 Leader 会定期向 PD 汇报 Region 的状态信息

每个 Raft Group 的 Leader 和 PD 之间存在心跳包，用于汇报这个 Region 的状态。

PD 不断的通过这两类心跳消息收集整个集群的信息，再以这些信息作为决策的依据。

### 调度的策略

- 一个 Region 的副本数量正确
- 一个 Raft Group 中的多个副本不在同一个位置
- 副本在 Store 之间的分布均匀分配
- Leader 数量在 Store 之间均匀分配
- 访问热点数量在 Store 之间均匀分配
- 各个 Store 的存储空间占用大致相等
- 控制调度速度，避免影响在线服务

### 调度的实现

PD 不断地通过 Store 或者 Leader 的心跳包收集整个集群信息，并且根据这些信息以及调度策略生成调度操作序列。每次收到 Region Leader 发来的心跳包时，PD 都会检查这个 Region 是否有待进行的操作，然后通过心跳包的回复消息，将需要进行的操作返回给 Region Leader，并在后面的心跳包中监测执行结果。

注意这里的操作只是给 Region Leader 的建议，并不保证一定能得到执行，具体是否会执行以及什么时候执行，由 Region Leader 根据当前自身状态来定。
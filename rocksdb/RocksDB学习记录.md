[TOC]

## 待学习的博客

[Rocksdb Compaction 源码详解（一）：SST文件详细格式源码解析_z_stand的博客-CSDN博客_rocksdb sst文件](https://blog.csdn.net/Z_Stand/article/details/106959058)

[论文笔记 - Optimizing Space Amplification in RocksDB - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/184974230)

[RocksDB Compaction源码分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/369386111)

[Dostoevsky: 一种更好的平衡 LSM 空间和性能的方式 - 简书 (jianshu.com)](https://www.jianshu.com/p/8fb8f2458253)

[宋昭 - 知乎 (zhihu.com)](https://www.zhihu.com/people/kernelmaker/posts)

## 待学习的论文

[Dostoevsky: Better Space-Time Trade-Offs for LSM-Tree Based Key-Value Stores via Adaptive Removal of Superfluous Merging (harvard.edu)](https://stratos.seas.harvard.edu/files/stratos/files/dostoevskykv.pdf)

# RocksDB零基础学习

## (一) What's RocksDB

[RocksDB零基础学习(一) What's RocksDB - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/153480392)

### 设计思想：冷热分离

新写入的“热数据”会保存在内存中，如果一段时间没有更新，冷数据会“下沉”到磁盘底层的“表层文件”，如果继续没有更新过这个问题，冷数据继续“下沉”到更底层的文件中。如果磁盘底层的冷数据被修改了，它又会再次进入内存，一段时间后又会被持久化刷回到磁盘文件的浅层，然后再慢慢往下移动到底层。

### Performance

1. it should be performant for fast storage and for server workloads
2. It should support efficient point lookups as well as range scans.
3. It should be configurable to support high random-read workloads, high update workloads or a combination of both.
4. Its architecture should support easy tuning of trade-offs for different workloads and hardware.

### B Tree

B树是一种专用的M阶树，可广泛用于磁盘访问。 M阶树顺序的B树最多可以有`m-1`个键和M个子树。 使用B树的主要原因之一是它能够在单个节点中存储大量键，并且通过保持树的高度相对较小来存储大键值。

排序M的B树包含M阶树的所有属性。 此外，它还包含以下属性。

- B树中的每个节点最多包含`m`个子节点。
- 除根节点和叶节点外，B树中的每个节点至少包含`m/2`个子节点。
- 根节点必须至少有`2`个节点。
- 所有叶节点必须处于同一级别。

### B+ Tree

B+树是B树的扩展，允许有效的插入，删除和搜索操作。

在B树中，键和记录都可以存储在内部节点和叶子节点中。 然而，在B+树中，记录(数据)只能存储在叶节点上，而内部节点只能存储键值。

B+树的叶节点以单链表的形式链接在一起，以使搜索查询更有效。

B+树用于存储无法存储在主存储器中的大量数据。 由于主存储器的大小总是有限的事实，B+树的内部节点(访问记录的键)存储在主存储器中，而叶节点存储在辅助存储器中。

B+树的内部节点通常称为索引节点。

### B树与B+树比较

| 编号 | B树                                                          | B+树                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | 搜索键无法重复存储。                                         | 可以存在冗余搜索键。                                         |
| 2    | 数据可以存储在叶节点以及内部节点中                           | 数据只能存储在叶节点上。                                     |
| 3    | 搜索某些数据是一个较慢的过程，因为可以在内部节点和叶节点上找到数据。 | 搜索速度相对较快，因为只能在叶节点上找到数据。               |
| 4    | 删除内部节点非常复杂且耗时。                                 | 删除永远不会是一个复杂的过程，因为元素将始终从叶节点中删除。 |
| 5    | 叶节点不能链接在一起。                                       | 叶节点链接在一起以使搜索操作更有效。                         |

### LSM Tree

适合于高频写入的同时，提供快速地查找，通过牺牲了部分读性能，用来大幅提高写性能。

这个设计思想的依据是：

- 内存的速度超磁盘1000倍以上。而读取的性能提升，主要还是依靠内存命中率而非磁盘读的次数。
- 写入不占用磁盘的IO，读取就能获取更长时间的磁盘IO使用权，从而也可以提升读取效率。

数据随机写操作（包括插入、修改、删除也是写）都在内存中进行，这样也一定程度地提成了写的性能。

## (二) Memtable & WAL(Write-ahead Log)

[RocksDB零基础学习(二) Memtable & WAL(Write-ahead Log) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/156831542)

RocksDB有三种基本文件格式是 Memtable/SST file/Log file。Memtable 是一种内存文件数据系统，新写数据会被写进 Memtable，部分请求内容会被写进 Log file。Log file 是一种有利于顺序写的文件系统。Memtable 的内存空间被填满之后，会有一部分老数据被转移到 SST file 里面，这些数据对应的 Log file 里的 log 就会被安全删除。SST file 中的内容是有序的。

### MemTable

RocksDB中MemTable的默认实现方式是SkipList，适用于范围查询和插入。但在不同场景，RocksDB也提供了另外两种实现方式：

- a vector memtable：适用于批量载入过程，每次新增元素都会追加到之前的元素后
- a prefix-hash memtable：适用于带有指定前缀的查询，插入和scan

### SkipList

[SkipList的原理与实现 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/33674267)

即跳表，设计初衷是作为替换平衡树的一种选择。

AVL树有着严格的O(logN)的查询效率，但是由于插入过程中可能需要多次旋转，导致插入效率较低，因而才有了在工程界更加实用的红黑树。但是红黑树有一个问题就是在并发环境下使用不方便，比如需要更新数据时，红黑树有个平衡的过程，在这个过程中会涉及到较多的节点，需要锁住更多的节点，从而降低了并发性能。而SkipList需要更新的部分比较少，锁的东西也更少，对并发性能的影响更小。

#### 原理及实现

跳表是在普通单向链表的基础上增加了一些索引，且这些索引是分层的，从而可以快速地查到数据。

- Node

  ```c++
  template<typename K, typename V>
  class SkipList;
  
  template<typename K, typename V>
  class Node {
  	// friend class：友元类，在一个类中指明其他的类（或者）函数能够直接访问该类中的private和protected成员
      // skipList类可以直接访问Node中的private成员
      friend class SkipList<K, V>;
  
  public:
  
      Node() {}
  
      Node(K k, V v);
  
      ~Node();
  
      K getKey() const;
  
      V getValue() const;
  
  private:
      K key;
      V value;
      Node<K, V> **forward; // 指向包含指向其他节点的指针的数组的指针
      int nodeLevel; // 该节点处于的level
  };
  
  template<typename K, typename V>
  Node<K, V>::Node(const K k, const V v) {
      key = k;
      value = v;
  };
  
  template<typename K, typename V>
  Node<K, V>::~Node() {
      delete[]forward;
  };
  
  template<typename K, typename V>
  K Node<K, V>::getKey() const {
      return key;
  }
  
  template<typename K, typename V>
  V Node<K, V>::getValue() const {
      return value;
  }
  ```

- 查找：从header出发，从高到低的level进行查找

  ```cpp
  template<typename K, typename V>
  Node<K, V> *SkipList<K, V>::search(const K key) const {
      Node<K, V> *node = header;
      for (int i = level; i >= 0; --i) {
          while ((node->forward[i])->key < key) { // 当当前node指向的下一个node的key值大于要查找的key值时，从当前node的下一级开始查找
              node = *(node->forward + i);
          }
      }
      node = node->forward[0];
      if (node->key == key) {
          return node;
      } else {
          return nullptr;
      }
  };
  ```

- 插入：关键在于找到合适的插入位置，即从所有小于待插入节点key值的节点中，找出最大的那个

  ```cpp
  template<typename K, typename V>
  bool SkipList<K, V>::insert(K key, V value) {
      Node<K, V> *update[MAX_LEVEL];
      Node<K, V> *node = header;
      for (int i = level; i >= 0; --i) {
          while ((node->forward[i])->key < key) {
              node = node->forward[i];
          }
          update[i] = node;
      }
      //首个结点插入时，node->forward[0]其实就是footer
      node = node->forward[0];
      //如果key已存在，则直接返回false
      if (node->key == key) {
          return false;
      }
      int nodeLevel = getRandomLevel();
      if (nodeLevel > level) {
          nodeLevel = ++level;
          update[nodeLevel] = header;
      }
      //创建新结点
      Node<K, V> *newNode;
      createNode(nodeLevel, newNode, key, value);
      //调整forward指针
      for (int i = nodeLevel; i >= 0; --i) {
          node = update[i];
          newNode->forward[i] = node->forward[i];
          node->forward[i] = newNode;
      }
      ++nodeCount;
  #ifdef DEBUG
      dumpAllNodes();
  #endif
      return true;
  };
  ```

- 移除

  ```cpp
  template<typename K, typename V>
  bool SkipList<K, V>::remove(K key, V &value) {
      Node<K, V> *update[MAX_LEVEL];
      Node<K, V> *node = header;
      for (int i = level; i >= 0; --i) {
          while ((node->forward[i])->key < key) {
              node = node->forward[i];
          }
          update[i] = node;
      }
      node = node->forward[0];
      //如果结点不存在就返回false
      if (node->key != key) {
          return false;
      }
      value = node->value;
      for (int i = 0; i <= level; ++i) {
          if (update[i]->forward[i] != node) {
              break;
          }
          update[i]->forward[i] = node->forward[i];
      }
      //释放结点
      delete node;
      //更新level的值，因为有可能在移除一个结点之后，level值会发生变化，及时移除可避免造成空间浪费
      while (level > 0 && header->forward[level] == footer) {
          --level;
      }
      --nodeCount;
  #ifdef DEBUG
      dumpAllNodes();
  #endif
      return true;
  };
  ```

### Immutable Memtable

所有的写操作都是在MemTable进行，当MemTable空间不足时，会创建一块新的MemTable来继续接收写操作，原先的内存将被标识为只读模式，等待被刷入SST。刷入时机有以下三个条件来确定：

- write_buffer_size ：一块MemTable的容量
- max_write_buffer_number：MemTable的最大存在数（active和immutable共享）
- min_write_buffer_number_to_merge：设置在刷入SST之前，最小可合并的immutable MemTable数

例如：

```text
write_buffer_size = 512MB;
max_write_buffer_number = 5;
min_write_buffer_number_to_merge = 2;
```

当写入速达到 16MB/s，每32s会创建一块新的memtable,每64s会有2块memtable合并，并刷入sst 。每一刷在512MB 到 1GB之间。为了防止刷入操作影响写性能，内存使用上限是5*512MB = 2.5GB。到达上限后，所有的写请求将会被阻塞。

在任何时候，一个CF（compact flash）中，只存在一块active MemTable和0+块immutable MemTable。

### WAL(Write-ahead Log)

每次数据被更新时，会同时写入内存表和WAL，WAL可用于发生故障后，恢复内存的数据。

## (三) Level Style Compaction

[RocksDB零基础学习(三) Level Style Compaction - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/156848134)

### Amplification

- **Write amplification** is the multiple of bytes written by the database to bytes changed by the user. Since some LSM trees rewrite unchanging data over time, write amplification can be high in LSM trees.
- **Read amplification** is how many bytes the database has to physically read to return values to the user, compared to the bytes returned. Since LSM trees may have to look in several places to find data, or to determine what the data’s most recent value is, read amplification can be high.
- **Space amplification** is how many bytes of data are stored on disk, relative to how many logical bytes the database contains. Since LSM trees don’t update in place, values that are updated often can cause space amplification.

### Compactions

- level间
- level0的kv在MemTable flush时
- SortedRun

### Compaction Algorithm

[LSM Tree的Leveling 和 Tiering Compaction - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/112574579)

#### Run的wiki解释

Each run contains data sorted by the index key. A run can be represented on disk as a single file, or alternatively as a collection of files with non-overlapping key ranges.

要点：sorted，non-overlapping key ranges

#### Leveled compaction

在Disk 里面维持了多级level的SStable，而且每层维持“**唯一一个**”Run（SortedRun）。

Level compaction目标就是维持每个level都保持住**one data sorted run**，所以每个level都可以和下一个level做compaction，同时很有可能会被上一个level做compaction。这样做好处就是level之间的compaction可以multithread来做（除了memory到level0），提高效率。RocksDB用的默认就是Leveled compaction

#### Tiered compaction

这种compaction的方法，可以保证每个sstable都是sorted，但不能保证每一层只有一个run。MemTables首先会不停flush到第一层很小的sstables，等到sstable数量够了（比如4个），compaction成一个sstable写到下一层，下一层sstable写满4个，再compact，如此以往。Cassandra默认的就是Tiered compaction。

#### 两者区别

- Write amplification：由于leveled 需要更频繁的compaction以保证每个level只有一个run，因此leveled 的write amplification更大
- Read amplification：由于tiered 中的每层不只有一个sorted run，因此每次查找需要读的次数更多，read amplification也更大
- Space amplification：更新更快的leveled 在space amplification上要更小

#### Leveled +Tiered

- 比起单独的两种算法，二者相结合后有较小的写放大和较小的空间放大
- Tiered compaction for the smaller levels and leveled compaction for the larger levels.
- 在rocksDB Level Style 是用的这种算法。比如memtable 可以通过配置max_write_buffer_number来设置多个sorted runs，但只有一个是可写的，其他都是只读。

### Compaction Styles

#### minor compaction 和 major compaction

- minor compaction：把memtable中的数据导出到SSTable文件中
- major compaction：合并不同层级的SSTable文件
- FIFO Style Compaction 是第三种compaction style，会直接删除旧数据，可用于缓存
- 另外，RocksDB支持自定义compaction方式

#### Minor compaction

按照immutable memtable中记录由小到大遍历，并依次写入一个level 0 的新建SSTable文件中，写完后建立文件的index 数据。

#### Major Compaction(leveled compaction)

- 在L0，不同的文件可能会出现相同的key（因此，每次Get()需要遍历L0的每个文件），但是L1和更高level的文件中，不同的文件不会出现相同的key。
- 除L0外，每个Level的数据按key的顺序分为一个个SST file。每个SST file内部的key都是有序的。
- 所有非0的Level都有target sizes，Compaction的目的就是把每层的数据控制在target size下。每一层的target size通常是指数递增的。
- 每一次compaction，会从Ln中取一部分文件与Ln+1的重叠文件进行合并。运行在不同Level或者不同key 范围的compaction可以**并发**执行。
- L0->L1的compaction是最棘手的：L0可能覆盖key的整个范围且key之间有重叠，当L0->L1运行时，会将L0层的所有文件全部merge到L1层，L1的所有file都参与进来，这样L1->L2的compaction就不能执行。如果L0->L1太慢，可能整个系统在长时间内都只存在这一个compaction操作，并且L0->L1是单线程任务。
- 为解决L0→L1的compaction操作会成为整个服务的性能瓶颈问题，RocksDB引入max_subcompactions，可将一个file 分块compaction。

### which level to compact

当多个level都达到了compaction时，需要计算level score来确定哪个level先compaction。
$$
level\_score = \frac{current\ level\ size}{max\ level\ size}
$$

### which file to compaction

- 通过level sore 选择发生compaction的level n
- 确定Level n+1
- 在Level n通过**优先级配置项**选择参与compaction的第一个文件 SST file - fk，加入**inputs**
- 将fk 周围的其他SST file也加入**inputs,**除非这个file 与 **inputs** 中的文件有“清晰”的边界
  - SST file里，存储的“key” 是**InternalKey**，而**InternalKey**是由UserKey+seqNo+key type构成
  - 所以，可能同一个user key 会以不同的**InternalKey**形式存储在不同的文件中。如果发生compaction，我们需要把这些user_key相同的**InternalKey**都加入**Inputs**
- 检查Ln+1中，与**Inputs** 重叠的SST file，是否有正在compaction的，如果有正在compaction的SST file，就尝试手动compaction，如果没有找到可以手动compaction的，就直接停止此次compaction。
- 在Ln+1 的重叠文件中，将符合条件的SST file加入**output_level_inputs。**
- 有个可选的优化项，尝试在不改变**output_level_inputs** 的SST file数的前提下，去增加Ln的**Input**数。

#### 优先级配置项

- kOldestSmallestSeqFirst：选择最晚更新（oldest update）的文件

  如果想在某一层选择参与compaction的SST file，选择更旧的文件，它的key range 可能会更小，引起的**output_level_inputs**文件可能会更少，也就间接减小了“写放大”。因此，如果写入场景中key的更新范围较为均匀，可以选择配置**options.compaction_pri = kOldestSmallestSeqFirst** 来减少写放大。

- kByCompensatedSize：选择删除标记最多的文件(Default)

  减少查询被删除的key造成的查询效率降低，提高空间效率

- 外部压缩：call DB::CompactFiles() 来压缩单个文件。

- kMinOverlappingRatio：选择与下一层level文件中覆盖最小的

## (四) Universal Style Compaction

[RocksDB零基础学习(四) -Universal Style Compaction - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/165137544)

universal compaction有个前置条件，就是sorted run的数量到达一定要到达**options.level0_file_num_compaction_trigger**，否则不会触发compaction。当达到先决条件后，以下四种情况都可以触发实际的compaction。

假设$R_1,R_2,R_{n-1},R_n$为n个sorted run，$R_1$有DB最新的更新，而$R_n$有最老的DB更新。


### Compaction Triggered by Space Amplification

当***size amplification ratio*** 值大于**options.compaction_options_universal.max_size_amplification_percent / 100**时，所有的sorted runs 会被merge 成一个sorted run。
$$
size\ amplification\ ratio=\frac{\sum_{i=1}^{n-1}R_i}{R_n}
$$
这里可以看出，Universal Compaction的目标是，每次compaction，都希望把活跃数据都向Rn也就是最大的一个子集靠拢，理想状态下，RocksDB拥有一个全量active data的sorted runs。

RocksDB里options.compaction_options_universal.max_size_amplification_percent 的默认值为200，也就是说，将保持300%的active data。

### Compaction Triggered by Individual Size Ratio

size ratio的计算公式为：
$$
size\_ratio\_trigger=(100+options.compaction\_options\_universal.size\_ratio)/100
$$
通常 options.compaction_options_universal.size_ratio会被设置为0，那么size_ration_trigger默认为1。

从R1开始，如果 size(R2) / size(R1) <= *size ratio trigger* 那么（R1,R2）一起参与compaction，继续，如果 size(R3) / size(R1 + R2) <= *size ratio trigger*,那么R3会加入进来（R1,R2,R3）一起参与compaction,然后我们继续找R4,R5,直到某一个sorted run不满足这个条件。

需要注意的是，只有当输入的sorted runs至少达到options.compaction_options_universal.min_merge_width时，才会触发合并，而sorted runs的上限不超过options.compaction_options_universal.max_merge_width。

### Compaction Triggered by number of sorted runs

如果sorted runs的数量达到了options.level0_file_num_compaction_trigger，但是没有触发前面提到的size amplification 或者space amplification trigger。那么RocksDB将尝试去将这些sorted runs compaction，以使最终的数量小于等于options.level0_file_num_compaction_trigger。

### Compaction triggered by age of data

对于Universal Compaction,基于时间的compaction策略是最高优先级的，如果我们尝试compaction，一定会先检查时间条件，如果有文件存在的时长大于options.periodic_compaction_seconds，RocksDB将会从旧到新来挑选sorted runs 来compaction，直到某个更新的sorted runs 正在被compaction，这些文件将会被压缩到最底层。

## (五) SSTable（Sorted Sequence Table）

[RocksDB零基础学习(五) SSTable（Sorted Sequence Table） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/165399524)

### SSTable定义

A block, once written to storage, is never modified.SSTable自创建后，要么被销毁，要么永不改变。

### 默认结构

![preview](https://pic2.zhimg.com/v2-2278b5624357bfc48d60c05855337da1_r.jpg)

```text
<beginning_of_file>
[data block 1]
[data block 2]
...
[data block N]
[meta block 1: filter block]              (see section: "filter" Meta Block)
[meta block 2: index block]
[meta block 3: compression dictionary block]  (see section: "compression dictionary" Meta Block)
[meta block 4: range deletion block]      (see section: "range deletion" Meta Block)
[meta block 5: stats block]                   (see section: "properties" Meta Block)
...
[meta block K: future extended block]  (we may add more meta blocks in the future)
[metaindex block]
[Footer]                               (fixed size; starts at file_size - sizeof(Footer))
<end_of_file>
```

#### data block

```cpp
struct Entry {
  varint sharedKeyLength; // 与前一条记录key共享部分的长度，为0则表示该 Entry 是一个重启点
  varint unsharedKeyLength; // 与前一条记录key不共享部分的长度
  varint valueLength; // value长度
  byte[] unsharedKeyContent; // 与前一条记录key非共享的内容
  byte[] valueContent; // value内容
}


struct DataBlock {
  Entry[] entries;
  int32 [] restartPointOffsets;
  int32 restartPointCount;
}
```

- 第一部分(Entry)用来存储key-value数据。由于sstable中所有的key-value对都是严格按序存储的，用了节省存储空间，并不会为每一对key-value对都存储完整的key值，而是存储与上一个key非共享的部分，避免了key重复内容的存储。
- 每间隔若干个key-value对，将为该条记录重新存储一个完整的key。重复该过程（默认间隔值为16），每个重新存储完整key的点称之为Restart point。Restart point的目的是在读取sstable内容时，加速查找的过程。由于每个Restart point存储的都是完整的key值，因此在sstable中进行数据查找时，可以首先利用restart point点的数据进行键值比较，以便于快速定位目标数据所在的区域；当确定目标数据所在区域时，再依次对区间内所有数据项逐项比较key值，进行细粒度地查找。
- 一个 DataBlock 的默认大小只有 4K 字节，这里的4K 字节，并不是说它的严格大小，而是在追加完最后一条记录之后发现超出了 4K 字节，这时就会再开启一个 DataBlock。这意味着一个 DataBlock 可以大于 4K 字节，如果 value 值非常大，那么相应的 DataBlock 也会非常大。DataBlock 并不会将同一个 Value 值分块存储。

#### meta block

##### BlockHandle 结构体

```text
offset:         varint64 //在sstable中的偏移量
size:           varint64 //数据长度
```

##### Bloom Filter

- 判断查询的key 在不在这个SST文件上,如果不存在，则不用查找这个datablock
- 默认只有加载SST file，bloom filter 才会载入内存,如果SST file被关闭，bloom filter 也会被释放，可以通过配置取消释放
- Bloom Filter和Index block可能占用大量内存，并且不算在block cache的配置中

##### Filter Block

```cpp
struct FilterEntry {
  byte[] rawbits;
}


struct FilterBlock {
  FilterEntry[n] filterEntries;
  int32[n] filterEntryOffsets;
  int32 offsetArrayOffset;
  int8 baseLg;  // 分割系数
}
```

- FilterOffset：再进一步获得相应的布隆过滤器位图数据
- baseLg：base指的是一个数值，大小为2KB，以对数形式存储，则baseLg约等于11，存储只占一个字节。默认值表示每2KB的数据，创建一个新的过滤器来存放过滤数据。这里的 2K 字节的间隔是严格的间隔，这样才可以通过 DataBlock 的偏移量和大小来快速定位到相应的布隆过滤器的位置

#### Index block

分区索引是将原索引快分成一个个小的索引快，然后在它们上层加了一层索引。在执行filter/index 查询时，先走上层索引，根据定位再加载二级索引。

#### Metaindex

记录了每个meta block的信息，同样每份都是kv结构，key 就是meta block 的name，value是一个BlockHandle类型的指针，指向了真实的位置。

#### Footer 

Kv 结构存储了当前sst 文件的各种属性，指明了 MetaIndexBlock 和 IndexBlock的位置，进而找到 MetaBlock 和 DataBlock。是一种定长的格式，为48个字节（旧）或53字节（新），读取 SST文件的时候，就是从文件末尾，固定读取这48或53字节，进而得到了 Footer 信息。

```cpp
//  见format.h
class Footer {
public:
  Footer() : Footer(kInvalidTableMagicNumber, 0) {}
  Footer(uint64_t table_magic_number, uint32_t version);
......
private:
  uint32_t version_;
  ChecksumType checksum_;
  BlockHandle metaindex_handle_;
  BlockHandle index_handle_;
  uint64_t table_magic_number_ = 0;
};
```

# RocksDB系列

## 一：RocksDB基础和入门

[RocksDB系列一：RocksDB基础和入门 - 简书 (jianshu.com)](https://www.jianshu.com/p/061927761027)

### 简介

RocksDB深度支持各种配置，可以在不同的生产环境（纯内存、Flash、hard disks or HDFS）中调优，支持不同的数据压缩算法、和生产环境debug的完善工具。

RocksDB的主要设计点是在快存和高服务压力下性能表现优越，所以该DB需要充分挖掘Flash和RAM的读写速率。

RocksDB需要支持高效的point lookup（点查询）和range scan（范围查询）操作，需要支持配置各种参数在高压力的随机读、随机写或者二者流量都很大时性能调优。

#### HDFS

[HDFS - 简书 (jianshu.com)](https://www.jianshu.com/p/f1e785fffd4d)

即Hadoop Distributed File System，是Hadoop项目的核心子项目，是分布式计算中数据存储管理的基础，是基于流数据模式访问和处理超大文件的需求而开发的，可以运行于廉价的商用服务器上。它所具有的高容错、高可靠性、高可扩展性、高获得性、高吞吐率等特征为海量数据提供了不怕故障的存储，为超大数据集（Large Data Set）的应用处理带来了很多便利。

### 特性

[RocksDB -- 特性 | GuKaifeng's Blog](https://gukaifeng.cn/posts/rocksdb-te-xing/#8-Multi-Threaded-Compactions)

### Column Family（列族）

[RocksDB-Column Family(列族)_一书一茶一世界的博客-CSDN博客_rocksdb 列族](https://blog.csdn.net/weixin_43763259/article/details/120847118)

在RocksDB 3.0中加入了Column Family特性，加入这个特性之后，每一个KV对都会关联一个Column Family,其中默认的Column Family是 “default”. Column Family主要是提供给RocksDB一个逻辑的分区.从实现上来看不同的Column Family共享WAL，而都有自己的Memtable和SST.这就意味着我们可以很快速已经方便的设置不同的属性给不同的Column Family以及快速删除对应的Column Family.

### Update

Put()：把一对K-V数据写入DB，如果K已存在，则已有的V会被新的V覆盖。

Write()：将多对K-V数据写入DB，保证要么所有的K-V对都写入DB，要么一个都不写入。

### Get

Get()：从DB中查询key对应的value，MultiGet()：批量查询。（DB中的所有数据都是按照key有序存储，其中key的compare方法可以用户自定义。）

### Iterator

Iterator()提供给用户RangeScan功能，首先Seek()到一个特定的key，然后从这个点开始遍历。当执行Iterator时，用户看到的是一个时间点的一致性视图。

### Snapshot

Snapshot()：可以创建数据库在某一个时间点的快照。Get和Iterator接口也可以执行在某一个Snapshot上。

### Iterator VS Snapshot

某种意义上，Iterator和Snapshot提供了DB在某个时间点的一个一致性视图，但是其实现原理却不一样。快速短期/前台的**scan**操作比较适合用**Iterator**，长期/后台操作适合用**Snapshot**。

当使用Iterator时，会对数据库相应时间点的所有底层文件增加引用计数，直到Iterator结束或者释放了引用计数后，这些文件才允许被删除。

Snapshot不关注数据文件是否被删除的问题，Compation进程会感知Snapshot的存在，会保证对应视图的数据不会被删除。当实例重启时，Snapshot会丢失，这是因为RocksDB不会持久化Snapshot相关数据。

### Transaction（事务）

**[【RocksDB源码分析】TransactionDB介绍 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/39427559)**

RocksDB的Transaction分为两类：Pessimistic和Optimistic，类似悲观锁和乐观锁的区别，Pessimistic Transaction的冲突检测和加锁是在事务中每次写操作之前做的（commit后释放），如果失败则该操作失败；OptimisticTransaction不加锁，冲突检测是在commit阶段做的，commit时发现冲突则失败。

### Prefix Iterator

[RocksDB. Prefix Seek源码分析 - 简书 (jianshu.com)](https://www.jianshu.com/p/9848a376d41d)

大部分的LSM引擎都不支持高效的RangeScan操作，这是由于执行RangeScan操作时都要访问所有的数据文件导致。但是大部分用户并不仅仅是完全scan所有的数据，相反，很多情况下仅仅需要按照key的前缀字符串区遍历。RocksDB根据这些应用场景，优化了对应的底层实现。用户可以prefix_extractor来声明一个key_prefix，然后RocksDB为每一个key_prefix存储相应的blooms。配置了key_prefix的Iterator操作可以通过对应的bloom bits来避免检索不含有特定key prefix的数据文件，依次可以提高Iterator性能。

### Persistence

RocksDB有事物日志（类似WAL），所有的写操作首先写入内存表内，然后可选地写入到事物日志中。当DB重启时会重新执行事物日志中的所有操作，然后恢复到特定的数据状态。

事物日志数据可以与DB数据文件配置成不同的目录下，这种情况适用于将数据文件写到一致性、性能高的快存中，同时可以将事物日志保存在读写性能相对比较慢的持久化存储上来保证数据的安全性。

当写数据时可以配置WriteOption,来支持是否将写操作记录在事物日志中或者当用户执行commit时是否需要执行事物日志记录的sync(同步)操作。

### Fault Torlerance

RocksDB通过checksum来检测磁盘数据损坏。每个sst file的数据块（4k-128k）都有相应的checksum值。写入存储的数据块内容不允许被修改。

### Multi-Threaded Compaction

当用户重复写入一个key时，在DB中会存在这个key的多个value，compaction操作就是来删除这个key的冗余数据。当一个key被删除时，compation也可以用来真正执行这个底层数据的删除工作，如果用户配置合适的话，compation操作可以多线程执行。

### Mitigate Stall

[Tuning RocksDB - Write Stalls - 简书 (jianshu.com)](https://www.jianshu.com/p/a2892a161a7b)

### Full Backup and Replication

RocksDB 提供了备份 API BackupEngine，RocksDB 本身不是复制的，但它提供了一些帮助功能，使用户能够在RocksDB上实现复制系统。

### Block Cache

 RocksDB使用LRU cache提供block的读服务。block cache partition为两个独立的cache，其中一块可以cache未压缩RAM数据，另一块cache 压缩RAM数据。

### Table Cache

Table cache缓存了所有已打开的文件句柄，这些文件都是sstfile。用户可以设置table cache的最大值。

### Merge Operator

MergeOperator定义了几个方法来告诉RocksDB应该如何在已有的数据上做增量更新。这些方法(PartialMerge、FullMerge)可以组成新的merge操作。

[TOC]

# levelDB学习记录

## 博客

[leveldb源码阅读系列 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/80684560)

## LevelDB源码分析

### 1.起步

[LevelDB源码解析1.起步 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/34657032)

### 2.Visual Studio Code

[LevelDB源码解析2. Visual Studio Code - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/34657882)

### 3.基本思路

[LevelDB源码解析3. 基本思路 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/34658848)

#### LevelDB四个接口

```c++
class DB {
 public:
  virtual Status Put(
      const WriteOptions& options,
      const Slice& key,
      const Slice& value) = 0;

  virtual Status Delete(
      const WriteOptions& options,
      const Slice& key) = 0;

  virtual Status Write(
      const WriteOptions& options,
      WriteBatch* updates) = 0;

  virtual Status Get(
      const ReadOptions& options,
      const Slice& key,
      std::string* value) = 0;
}
```

四个接口：Put（单项写）、Delete、Write（批处理写，同时写成功/失败）、Get

#### 磁盘的特点

顺序写与顺序读速度比较快。随机写与随机读则需要磁盘寻道。因此单位时间内的读写(IOPS)比顺序的情况要差。一般要下降3倍左右。

#### 状态机

即State Machine，是有限状态自动机的简称，是现实事物运行规则抽象而成的一个数学模型。

##### 状态机的四个概念

- State：状态
- Event：事件，执行某个操作的触发条件或者口令
- Action：动作，事件发生以后要执行动作，编程时一个动作对应一个函数
- Transition：变换

#### 初步设计流程

1. 写WAL LOG
2. 更新内存，即是下图中的MemTable
3. 当内存size达到一定程度的时候。把Memtable变成不可变的内存块。
4. 把内存块与磁盘上的Block。这里叫做.sst文件进行合并。到这里内存的合并是顺序写的。
5. 磁盘上的block根据新旧先后分层。总是上面一层的与下面一层的合并。
6. 读的时候先查内存，内存中没有的时候，再顺次从Level0~LevelN里面的磁盘块内容。直到找到Key即返回相应的val。找不到说明不存在，返回NULL。

### 4.架构设计

[LevelDB源码解析4. 架构设计 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/34665791)

#### 写流程

1. 先写Log文件。也就是WAL Log。
2. 再把数据更新到Memtable。Memtable是一个内存实现的KV表。
3. 当Memtable size超过一定限制的时候，把Memtable整理成一个ImuTable。也就是不可更改的KV表。
4. 把不可更改的KV表写入到磁盘。然后释放相应的Log文件的空间。
5. 当Level0 SST文件大小太多的时候，Level 0 SST会与后面的层级进行合并，保证有序性。
6. Level 1 SST与更高的层次的合并也是这样。

#### 读流程

1. 先去Memtable中查找相应的键值，看一下是否存在。如果存在。那么返回相应的Value。这个时候读是最快的。
2. 不过运气一般都没有那么好。读的时候数据不一定在这个活跃的内存里面。这个时候就需要去ImuTable这个不可变的内存块里面碰运气了。如果运气好，如果能够直接找到。那么这个时候速度还算是够快的。因为都是在内存里面进行操作。
3. 如果在Memtable和imuTable里面都没有找到。那么这个时候就需要去磁盘块里面查找了。
4. 从数据的新旧程度上来说。Level 0里面的数据块是最新的。Level 1其次。所以在查找的时候，也是照着这个顺序查找。一个一个翻文件，如果翻到了就返回。如果没有翻到，那么就只能说读取失败。

#### Manifest文件

Manifest文件中记录了 LevelDB 中所有层级中的表、每一个 SSTable 的 Key 范围和其他重要的元数据，以日志的格式存储，所有对文件的增删操作都会追加到这个日志中。

#### Current文件

记载当前的Manifest文件名。

### 5.数据完整性

[LevelDB源码解析5. 数据完整性 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/34671492)

数据的完整性是说：当一个数据的状态要从a=X改变到a=Y的过程中时，要么成功，要么失败，不会出现a=X.5这种中间状态。

#### LOG对事务的支持

通常意义义上的日志文件是在LOG中。如果文件名是${number}.log，那么这种文件在leveldb中就是用来支持事务的，里面写入的就是各种各样的结构体。

当memtable中还有足够空间的情况下，先写WAL LOG，再更新到内存，最后返回客户端说更新成功。

为了保证数据的安全性，在合并的时候，不允许修改已有的SSTable文件，只允许生成新的文件。

简单代码描述如下：

```c++
const SSTable files{Range1, Range2, Range3};
const CompationOperations compact(Range1, Range2, Range3);
SSTable newFiles = files + compact;
```

为了保证数据的完整性，需要把compact写WAL LOG，写入成功后，再生成newfiles。

针对SSTable的WAL LOG叫Manifest。

### 6.版本概念

[LevelDB源码解析6. 版本概念 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/34674504)

因为LevelDB是支持多线程的，也就是说，当合并操作在读这三个sst文件的时候，其他的读操作也有可能落在这三个文件上。所以，删除文件的时候，需要等到这三个sst文件的引用ref = 0的时候才可以。

保留原有的文件，是为了支持合并时正在进行的读请求。

#### 版本控制

- 旧有的文件组成一个版本Version。

- 合并操作称之为VersionEdit。

- 旧的版本与新的版本可以共存。当新版本完成的之后。用户发起来的请求就到新的版本上。这个新的版本就叫current。

- 假设Version1在生成Version2的时候，Version1正在服务很多读请求。Version1自然不能丢。

- - 当Version2生成好了之后，Version2也开始服务很多读请求。
  - Version2到达某一时刻，也会开始进行合并，生成Version3。由于Version2还在服务读请求。自然不能丢弃。此时正在工作的整体就是{version1, version2, version3}，这个服务于当前DB的Version的集合，就称之为VersionSet。

- 当某个旧的Verions不再被读的时候。这些旧的Version就会被删除了。

### 7.日志格式

[LevelDB源码解析7. 日志格式 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35134533)

log_writer.h log_writer.cc

#### Slice和Record

结构体：

```C++
struct Slice {
    char *data_;
    size_t size_;
};
```

LevelDB主要想解决的问题就是想尽量地把随机写改成顺序写。如果过来的Slice都是非常小，那么顺序写的想法也就无法实施。所以其中一种办法是把多个slice放到一个block里面压缩好，写入到文件里面。在LevelDB里面设定一个block的大小是32KB。每次写入文件的时候，就可以变成批量写。这个时候基本可以认为是顺序的。

由于每个slice的长度都是不一样的。那么在读取的时候是非常难读取的。所以还需要加一些Metadata信息。

```C++
struct header {
    uint32_t crc_;        校验
    uint8_t length_low_;  长度的低8位
    uint8_t length_high_; 长度的高8位
    uint8_t type_;        数据的类型
};
struct Record {
    unsigned char header_[7];
    slice data_;
};
```

#### 关于Block的对齐

情况：

- 刚好对齐
- 一个block里面刚好放下多个record且刚好对齐
- 超大record，需要多个block
- 对不齐
  - 如果余下的空间已经不足以放置一个Record的header，这个余下的空间就用0来填。
  - 如果余下的空间刚好可以放置一个Record的header，那就放置下一个Record的header，后面的数据写到接下来的一个block里。接下来的Block的开头是一个新的header，前面7个bytes填写的header，里面记录的数据区的长度就是0。
  - 如果余下的空间比Record的header还多，那就直接用来填充数据，下一个block的开头就是一个header。

#### 关于Record的切分

一个Record与Block的关系可以是下面这三种：

- 一个Record刚好在一个Block里面
- 一个Record被分到两个Block里面
- 一个Record被切分到多个Block里面

因此：

- FULL表示这个Recorc没有跨越Block
- First表示这是Record的开头部分
- Middle表示这是Record的中间部分
- Last表示是一个Record的尾巴部分

对应的日志结构：

```c++
// log_format.h

enum RecordType {
  // Zero is reserved for preallocated files
  kZeroType = 0,

  kFullType = 1,

  // For fragments
  kFirstType = 2,
  kMiddleType = 3,
  kLastType = 4
};
static const int kMaxRecordType = kLastType;

static const int kBlockSize = 32768; //32KB

// Header is checksum (4 bytes), length (2 bytes), type (1 byte).
static const int kHeaderSize = 4 + 2 + 1;
```

#### EmitPhysicalRecord

写入文件的时候，是以Record为单位，而不是以Block为单位。

```c++
Status Writer::EmitPhysicalRecord(RecordType t, const char* ptr, size_t n) {
  // 根据前面的代码 fragment_length = min(avail, left)
  // 这里加上这个限制，应该是为了防止后面
  // block_offset_ + kHeaderSize + n溢出
  assert(n <= 0xffff);  // Must fit in two bytes 0xffff：65535
  // block_offset_是类成员变量。记录了在一个Block里面的偏移量。
  // block_offset_一定不能溢出。
  assert(block_offset_ + kHeaderSize + n <= kBlockSize);

  // leveldb是一种小端写磁盘的情况
  // LevelDB使用的是小端字节序存储，低位字节排放在内存的低地址端
  // Format the header
  // buf前面那个int是用来存放crc32的。
  char header[kHeaderSize];
  // 写入长度: 这里先写入低8位
  header[4] = static_cast<char>(n & 0xff);
  // 再写入高8位
  header[5] = static_cast<char>(n >> 8);
  // 再写入类型
  header[6] = static_cast<char>(t);

  // Compute the crc of the record type and the payload.
  // 这里是计算header和数据区的CRC32的值。具体过程不去关心。
  uint32_t data_crc = crc32c::Extend(type_crc_[t], ptr, n);
  data_crc = crc32c::Mask(data_crc); // Adjust for storage
  EncodeFixed32(header, data_crc);

  // Write the header and the payload
  Status s = dest_->Append(Slice(header, kHeaderSize));
  if (s.ok()) {
    s = dest_->Append(Slice(ptr, n));
    // 当写完一个record之后，这里就立马flush
    // 但是有可能这个slice并不是完整的。
    if (s.ok()) {
      s = dest_->Flush();
    }
  }
  // 在一个block里面的写入位置往前移。
  block_offset_ += kHeaderSize + n;
  return s;
}
```

##### assert断言

assert宏的原型定义在<assert.h>（C标准库）中，其作用是先计算表达式expression的值为假(即为0),那么它就先向stderr打印一条出错信息，然后通过条用abort来终止程序。使用assert的缺点是，频繁的调用会极大的影响程序的性能，增加额外的开销。

#### AddRecord

AddRecord的主要功能，就是把需要写入文件的Slice，切分成一个一个的record，然后顺序写入到文件中。

```c++
Status Writer::AddRecord(const Slice& slice) {
  const char* ptr = slice.data();
  size_t left = slice.size();

  // Fragment the record if necessary and emit it.  Note that if slice
  // is empty, we still want to iterate once to emit a single
  // zero-length record
  Status s;
  // begin是从数据的头开始写的么？

  bool begin = true;
  do {
    // 余下还需要多少才能填满一个块。
    const int leftover = kBlockSize - block_offset_;
    assert(leftover >= 0);

    // 如果剩下的空间已经很小了。连个头部都放不下了。
    // 这里需要分两种case.
    // 1. leftover == 0
    // 2. 0 < leftover < kHeaderSize
    if (leftover < kHeaderSize) {
      // Switch to a new block
      // Case 2. 如果不等于0，那么还是需要把剩下的空间用0来填满的。
      if (leftover > 0) {
        // Fill the trailer (literal below relies on kHeaderSize being 7)
        assert(kHeaderSize == 7);
        dest_->Append(Slice("\x00\x00\x00\x00\x00\x00", leftover));
      }
      // Case 1. 如果余下的等于0，那么也就什么都不用做了

      // 最后：
      // 移动到一个新的block的头上
      block_offset_ = 0;
    }

    // 这里有个恒等不变式
    // A. 也就是余下的空间必须要能够放一个header.
    // kBlockSize - block_offset = newOverLeft
    // newOverLeft - kHeaderSize >= 0
    // Invariant: we never leave < kHeaderSize bytes in a block. 
    assert(kBlockSize - block_offset_ - kHeaderSize >= 0);

    // 这里不能直接换掉上面的assert
    // 这里用的是size_t，如果换到上面，恒等不变式是一直成立的。
    // avail指的就是当前这个block里面的可用空间
    // 如果直接看这里的可用空间，就是kBlockSize - 0 - kHeaderSize
    // 所以也只是把当前record的header占用的空间扣掉了。
    const size_t avail = kBlockSize - block_offset_ - kHeaderSize;
    // left指的是sliceSizeToWrite.也就是dataToWrite
    // fragment_length指的就是后面需要写的数据量。主要是指当前这个block
    //    - 应该是在需要写入的数据量， 当前余下空间里面取最小值。
    const size_t fragment_length = (left < avail) ? left : avail;

    RecordType type;
    // 根据能写的数据的情况，来决定当前的这个record的类型。
    const bool end = (left == fragment_length);

    // 如果是从头开始写的，并且又可以直接把slice数据写完。
    // 那么肯定是fullType.
    if (begin && end) {
      type = kFullType;

    // 不能写完，但是是从头开始写
    } else if (begin) {
      type = kFirstType;

    // 不是从头开始写，但是可以把数据写完
    } else if (end) {
      type = kLastType;

    // 不能从头开始写，也不能把数据写完。
    } else {
      type = kMiddleType;
    }

    // 这里提交一个物理的记录
    // 注意：可能这里并没有把一个slice写完。
    s = EmitPhysicalRecord(type, ptr, fragment_length);
    // 移动写入指针。
    ptr += fragment_length;
    // 需要写入的数据相应减少
    left -= fragment_length;
    // 当然也不是从头开始写了。因为已经写过一次了。
    begin = false;
  } while (s.ok() && left > 0);
  return s;
}
```

### 8.读取日志

[LevelDB源码解析8. 读取日志 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35188065)

log_reader.h log_reader.cc

#### initial_offoset_

当需要读Slice的时候。不会每次都打开文件从头开始读，那么就需要给定一个开始读的位置，这个位置就叫做initial_offset_。

跳过块的代码如下：

```c++
bool Reader:: SkipToInitialBlock() {
  // 块中偏移
  const size_t offset_in_block = initial_offset_ % kBlockSize;
  // 需要跳过的块的位置
  // 这个变量的意思是说，后面在读的时候，要读的块的开头地址是什么？
  // uint64_t start_read_block_location = xx. 
  uint64_t block_start_location = initial_offset_ - offset_in_block;

  // Don't search a block if we'd be in the trailer
  // 如果给定的初始位置的块中偏移
  // 刚好掉在了尾巴上的6个bytes以内。那么
  // 这个时候，应该是需要直接切入到下一个block的。
  if (offset_in_block > kBlockSize - 6) {
    block_start_location += kBlockSize;
  }
  // 注意end_of_buffer_offset的设置是块的开始地址。
  end_of_buffer_offset_ = block_start_location;

  // Skip to start of first block that can contain the initial record
  if (block_start_location > 0) {
    Status skip_status = file_->Skip(block_start_location);
    if (!skip_status.ok()) {
      ReportDrop(block_start_location, skip_status);
      return false;
    }
  }

  return true;
}
```


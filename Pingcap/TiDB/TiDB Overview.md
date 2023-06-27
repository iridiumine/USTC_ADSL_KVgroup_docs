# TiDB Overview

## Architecture

### Overview

- An distributed scalable hybrid transactional and analytical processing (**HTAP**) database
- The open source implementation of **Google Spanner and Google F1**
- <img src="C:\Users\31795\AppData\Roaming\Typora\typora-user-images\image-20230627155037965.png" alt="image-20230627155037965" style="zoom:67%;" />

> **Google F1 / Spanner**
>
> Spanner is a distributed SQL database management and storage service developed by Google. It provides features such as global transactions, strongly consistent reads, and automatic multi-site replication and failover. Spanner is used in Google F1, the database for its advertising business Google Ads.

### Features

- Compatible with **MySQL** protocol
- Separate **SQL** and **KV** layers
- Using **Raft** for consistency and scaling
- Without a distributed file system

## Cluster Overview

### Components

- TiDB
- TiKV
- PD
- <img src="C:\Users\31795\AppData\Roaming\Typora\typora-user-images\image-20230627164351358.png" alt="image-20230627164351358" style="zoom:67%;" />

## TiKV Overview

- Each region stores a continuous range of data
  - The replicas of a region form a Raft Group
- <img src="C:\Users\31795\AppData\Roaming\Typora\typora-user-images\image-20230627164550548.png" alt="image-20230627164550548" style="zoom:67%;" />

## Technology Stack

- Relevant technologies
  - Spanner (Large-scale distributed database)
  - F1 (Large-scale distributed SQL)
  - Raft (Consensus algorithm)
  - Percolator (Distributed transaction)
  - RocksDB (Local key-value storage)
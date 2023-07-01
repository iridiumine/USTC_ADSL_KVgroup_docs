# pd-ctl region 指令结果解析

## node8上查看regions的指令

```
# distkv @ node8 in ~/jk/pd/bin on git:hot-sch-fluct x [16:19:18] 
$ ./pd-ctl -u 127.0.0.1:2481 region > ~/zsy/regions.txt
(base) 
```

## regions.txt的属性解析

- 例：

  ```
  {"count":2149,"regions":[{"id":26101,"start_key":"7480000000000000FF555F698000000000FF0000040158503235FF32313638FF353337FF0000000000FA0380FF0000000000000103FF8000000000000001FF0380000000000000FF010380000000037EFF14BB000000000000F9","end_key":"7480000000000000FF555F698000000000FF0000040158503235FF35353330FF333430FF0000000000FA0380FF0000000000000103FF8000000000000001FF0380000000000000FF01038000000003B3FFC5E5000000000000F9","epoch":{"conf_ver":5,"version":100},"peers":[{"id":26102,"store_id":1,"role_name":"Voter"},{"id":26103,"store_id":5,"role_name":"Voter"},{"id":26104,"store_id":4,"role_name":"Voter"}],"leader":{"id":26102,"store_id":1,"role_name":"Voter"},"written_bytes":0,"read_bytes":0,"written_keys":0,"read_keys":0,"approximate_size":102,"approximate_keys":958499},...,]}
  ```

- pd-ctl官方文档：[PD Control 使用说明](https://docs.pingcap.com/zh/tidb/stable/pd-control)

- TiDB SQL 语句：[SHOW TABLE REGIONS](https://docs.pingcap.com/zh/tidb/stable/sql-statement-show-table-regions)

  - `APPROXIMATE_SIZE(MB)`：估算的 Region 的数据量大小，单位是 MB。
  - `PEERS`：Region 的所有副本（replica）。
  - `LEADER`：Region 的 Leader 。

- TiDB 集群管理常见问题：[TiDB 集群管理常见问题](https://docs.pingcap.com/zh/tidb/stable/manage-cluster-faq)

  - `Approximate_Size` 表示压缩前表的单副本大小，该值为估算值，并非准确值。

- [TIKV_REGION_STATUS](https://docs.pingcap.com/zh/tidb/stable/information-schema-tikv-region-status)

  - `REGION_ID`：Region 的 ID。
  - `START_KEY`：Region 的起始 key 的值。
  - `END_KEY`：Region 的末尾 key 的值。
  - `EPOCH_CONF_VER`：Region 的配置的版本号，在增加或减少 peer 时版本号会递增。
  - `EPOCH_VERSION`：Region 的当前版本号，在分裂或合并时版本号会递增。
  - `WRITTEN_BYTES`：已经往 Region 写入的数据量 (bytes)。
  - `READ_BYTES`：已经从 Region 读取的数据量 (bytes)。
  - `APPROXIMATE_SIZE`：Region 的近似数据量 (MB)。
  - `APPROXIMATE_KEYS`：Region 中 key 的近似数量。

- TiKV监控指标：[TiKV 监控指标详解](https://docs.pingcap.com/zh/tidb/stable/grafana-tikv-dashboard)

- `"role_name":"Voter"`

  - [浅谈分布式存储之raft](https://zhuanlan.zhihu.com/p/114221938)
    - Raft节点有3种角色：
      - **Leader**：处理客户端读写、复制Log给Follower等。
      - **Candidate**：竞选新的Leader(由Follower超时转换得来)。
      - **Follower**：不发送任何请求，完全被动响应Leader、Candidate的RPC。
    - 节点的**Learner**状态：
      - 当集群成员变更时，新加入的节点为Learner状态，Learner状态的节点不算在Quorum节点内，不能参与投票；
      - 只有Leader确定这个Learner节点接收完了Snapshot，可以正常同步Log了，才可能将其变成可以正常的节点。
  - [深入浅出etcd/raft —— 0x05 Raft成员变更](http://blog.mrcroxx.com/posts/code-reading/etcdraft-made-simple/5-confchange/)
    - voter即有投票权的角色，包括candidate、follower、leader，而不包括learner。

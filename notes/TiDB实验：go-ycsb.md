# TiDB实验：go-ycsb

## 安装ycsb

首先要保证本地 Go 版本不低于 1.16，然后下载编译：

```
git clone https://github.com/pingcap/go-ycsb.git
cd go-ycsb
make
```

TiDB处于up状态，使用display查看状态

```
$ tiup cluster display tidb-test
```

如果down掉，需要重新启动TiDB cluster

```
$ tiup cluster start tidb-test --init
```

执行命令加载包含 10 万条记录和 30 万次操作的工作负载：

```
$ ../go-ycsb load  tikv -P ./go-ycsb/workloads/workloada -p tikv.pd="127.0.1.1:2379" -p tikv.type="raw
" -p recordcount=100000 -p operationcount=300000
```

成功加载数据后，您可以启动工作负载：

```
$ ../go-ycsb run tikv -P ./go-ycsb/workloads/workloada -p tikv.pd="127.0.1.1:2379" -p tikv.type="raw" -p recordcount=100000 -p operationcount=300000
```

### <a id="workload">workload</a>

- A：Update heavy workload，读写均衡型，50%/50%，Reads/Writes
- B：Read mostly workload，读多写少型，95%/5%，Reads/Writes
- C：Read only，只读型，100%，Reads
- D：Read latest workload，读最近写入记录型，95%/5%，Reads/insert
- E：Short ranges，扫描小区间型，95%/5%，scan/insert
- F：Read-modify-write workload，读写入记录均衡型，50%/50%，Reads/insert

关于Insert，Update，Read和Scan四种操作的占比：

- 云原生数据库中关于Insert，Update，Read和Scan四种操作的占比取决于具体的数据库系统和应用场景。不同的数据库系统和应用场景下，这些操作的占比可能存在很大差异。 例如，对于一个以读为主的OLTP（联机事务处理）应用来说，Read操作的占比会比较高，而Insert和Update操作的占比会相对较低。而对于一个以数据分析为主的OLAP（联机分析处理）应用来说，Scan操作的占比可能会比较高。 因此，无法给出统一的答案。针对具体的数据库系统和应用场景，需要进行实际测试和评估，才能得出相应的数据操作占比。

### <a id="requestdistribution">requestdistribution</a>

- uniform：随机选择一个记录；
- sequential：按顺序选择记录；
- zipfian：根据 Zipfian 分布来选择记录。大致意思就是互联网常说的80/20原则，也就是20%的key，会占有80%的访问量；
- latest：和 Zipfian 类似，但是倾向于访问新数据明显多于老数据；
- hotspot：热点分布访问；
- exponential：指数分布访问；

### 本地默认测试

#### workloada

##### 执行命令加载包含1000条记录和1000次操作的工作负载：

```
$ ../go-ycsb load  tikv -P ./go-ycsb/workloads/workloada -p tikv.pd="127.0.0.1:2379" -p tikv.type="raw"
```

```
[2023/03/28 09:28:59.049 +08:00] [INFO] [client.go:392] ["[pd] create pd client with endpoints"] [pd-address="[127.0.0.1:2379]"]
[2023/03/28 09:28:59.058 +08:00] [INFO] [base_client.go:350] ["[pd] switch leader"] [new-leader=http://127.0.0.1:2379] [old-leader=]
[2023/03/28 09:28:59.058 +08:00] [INFO] [base_client.go:105] ["[pd] init cluster id"] [cluster-id=7215401675930685265]
[2023/03/28 09:28:59.058 +08:00] [INFO] [client.go:687] ["[pd] tso dispatcher created"] [dc-location=global]
***************** properties *****************
"readproportion"="0.5"
"tikv.pd"="127.0.0.1:2379"
"command"="load"
"workload"="core"
"operationcount"="1000"
"recordcount"="1000"
"requestdistribution"="uniform"
"dotransactions"="false"
"insertproportion"="0"
"tikv.type"="raw"
"scanproportion"="0"
"readallfields"="true"
"updateproportion"="0.5"
**********************************************
INSERT - Takes(s): 9.9, Count: 850, OPS: 85.7, Avg(us): 11716, Min(us): 2490, Max(us): 84543, 99th(us): 21215, 99.9th(us): 36575, 99.99th(us): 84543
Run finished, takes 11.728742652s
INSERT - Takes(s): 11.6, Count: 1000, OPS: 85.9, Avg(us): 11686, Min(us): 2490, Max(us): 84543, 99th(us): 21151, 99.9th(us): 36575, 99.99th(us): 84543
[2023/03/28 09:29:10.789 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 09:29:10.789 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

##### 成功加载数据后，启动工作负载：

```
$  ../go-ycsb run tikv -P ./go-ycsb/workloads/workloada -p tikv.pd="127.0.0.1:2379" -p tikv.type="raw"
```

```
[2023/03/28 09:33:47.780 +08:00] [INFO] [client.go:392] ["[pd] create pd client with endpoints"] [pd-address="[127.0.0.1:2379]"]
[2023/03/28 09:33:47.785 +08:00] [INFO] [base_client.go:350] ["[pd] switch leader"] [new-leader=http://127.0.0.1:2379] [old-leader=]
[2023/03/28 09:33:47.785 +08:00] [INFO] [base_client.go:105] ["[pd] init cluster id"] [cluster-id=7215401675930685265]
[2023/03/28 09:33:47.785 +08:00] [INFO] [client.go:687] ["[pd] tso dispatcher created"] [dc-location=global]
***************** properties *****************
"workload"="core"
"command"="run"
"readproportion"="0.5"
"dotransactions"="true"
"updateproportion"="0.5"
"requestdistribution"="uniform"
"recordcount"="1000"
"operationcount"="1000"
"scanproportion"="0"
"readallfields"="true"
"insertproportion"="0"
"tikv.pd"="127.0.0.1:2379"
"tikv.type"="raw"
**********************************************
Run finished, takes 6.10272506s
READ   - Takes(s): 6.0, Count: 474, OPS: 78.7, Avg(us): 456, Min(us): 231, Max(us): 1247, 99th(us): 803, 99.9th(us): 1247, 99.99th(us): 1247
UPDATE - Takes(s): 6.1, Count: 526, OPS: 86.8, Avg(us): 11170, Min(us): 2658, Max(us): 42847, 99th(us): 22143, 99.9th(us): 26175, 99.99th(us): 42847
[2023/03/28 09:33:53.888 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 09:33:53.889 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

##### 连接grafana

安装vscode remote ssh插件，配置远程连接后，打开terminal，在port中配置端口，之后在浏览器打开即可。

![image-20230328095708463](C:\Users\31795\AppData\Roaming\Typora\typora-user-images\image-20230328095708463.png)

用户名：admin

密码：19991023zsy

[使用 Grafana 监控 TiDB 的最佳实践 | PingCAP 文档中心](https://docs.pingcap.com/zh/tidb/stable/grafana-monitor-best-practices)

打开grafana之后在search中找到集群——tidb-test，已经配置好很多仪表盘，直接查看即可。

#### workloadb

```
INSERT - Takes(s): 9.9, Count: 868, OPS: 87.7, Avg(us): 11468, Min(us): 2648, Max(us): 99647, 99th(us): 20127, 99.9th(us): 29983, 99.99th(us): 99647
Run finished, takes 11.526204881s
INSERT - Takes(s): 11.4, Count: 1000, OPS: 87.5, Avg(us): 11483, Min(us): 2648, Max(us): 99647, 99th(us): 20127, 99.9th(us): 29983, 99.99th(us): 99647
[2023/03/28 10:38:44.917 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:38:44.917 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

```
Run finished, takes 953.213187ms
READ   - Takes(s): 0.9, Count: 946, OPS: 1049.6, Avg(us): 497, Min(us): 215, Max(us): 52447, 99th(us): 787, 99.9th(us): 1030, 99.99th(us): 52447
UPDATE - Takes(s): 0.9, Count: 54, OPS: 61.1, Avg(us): 8816, Min(us): 6584, Max(us): 17935, 99th(us): 14759, 99.9th(us): 17935, 99.99th(us): 17935
[2023/03/28 10:39:14.013 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:39:14.013 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

#### workloadc

```
INSERT - Takes(s): 9.9, Count: 902, OPS: 91.1, Avg(us): 11040, Min(us): 2626, Max(us): 97151, 99th(us): 18991, 99.9th(us): 36319, 99.99th(us): 97151
Run finished, takes 11.062968899s
INSERT - Takes(s): 11.0, Count: 1000, OPS: 91.2, Avg(us): 11018, Min(us): 2626, Max(us): 97151, 99th(us): 18991, 99.9th(us): 36319, 99.99th(us): 97151
[2023/03/28 10:40:51.733 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:40:51.733 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

```
Run finished, takes 552.928343ms
READ   - Takes(s): 0.5, Count: 1000, OPS: 2005.4, Avg(us): 545, Min(us): 316, Max(us): 54847, 99th(us): 779, 99.9th(us): 2003, 99.99th(us): 54847
[2023/03/28 10:40:59.900 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:40:59.900 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

#### workloadd

```
INSERT - Takes(s): 9.9, Count: 873, OPS: 88.2, Avg(us): 11390, Min(us): 2662, Max(us): 106111, 99th(us): 20191, 99.9th(us): 31295, 99.99th(us): 106111
Run finished, takes 11.458149009s
INSERT - Takes(s): 11.4, Count: 1000, OPS: 88.1, Avg(us): 11417, Min(us): 2662, Max(us): 106111, 99th(us): 20191, 99.9th(us): 33919, 99.99th(us): 106111
[2023/03/28 10:42:32.964 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:42:32.964 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

```
Run finished, takes 1.024819574s
INSERT - Takes(s): 0.9, Count: 54, OPS: 58.4, Avg(us): 9141, Min(us): 6360, Max(us): 28463, 99th(us): 17951, 99.9th(us): 28463, 99.99th(us): 28463
READ   - Takes(s): 0.9, Count: 946, OPS: 1008.8, Avg(us): 551, Min(us): 218, Max(us): 87743, 99th(us): 787, 99.9th(us): 925, 99.99th(us): 87743
[2023/03/28 10:42:48.398 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:42:48.398 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

#### workloade

```
INSERT - Takes(s): 9.9, Count: 884, OPS: 89.3, Avg(us): 11268, Min(us): 2482, Max(us): 124991, 99th(us): 19983, 99.9th(us): 96447, 99.99th(us): 124991
Run finished, takes 11.298860279s
INSERT - Takes(s): 11.2, Count: 1000, OPS: 89.3, Avg(us): 11256, Min(us): 2482, Max(us): 124991, 99th(us): 19983, 99.9th(us): 96447, 99.99th(us): 124991
[2023/03/28 10:43:20.086 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:43:20.086 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

```
Run finished, takes 961.179764ms
INSERT - Takes(s): 0.8, Count: 48, OPS: 57.3, Avg(us): 8270, Min(us): 6176, Max(us): 12375, 99th(us): 12375, 99.9th(us): 12375, 99.99th(us): 12375
SCAN   - Takes(s): 0.9, Count: 952, OPS: 1053.6, Avg(us): 584, Min(us): 233, Max(us): 58431, 99th(us): 835, 99.9th(us): 49055, 99.99th(us): 58431
[2023/03/28 10:43:33.662 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:43:33.662 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

#### workloadf

```
INSERT - Takes(s): 9.9, Count: 887, OPS: 89.5, Avg(us): 11232, Min(us): 2636, Max(us): 94527, 99th(us): 23375, 99.9th(us): 48607, 99.99th(us): 94527
Run finished, takes 11.146590589s
INSERT - Takes(s): 11.1, Count: 1000, OPS: 90.5, Avg(us): 11104, Min(us): 2518, Max(us): 94527, 99th(us): 23215, 99.9th(us): 48607, 99.99th(us): 94527
[2023/03/28 10:43:59.366 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:43:59.366 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

```
Run finished, takes 5.848750949s
READ   - Takes(s): 5.8, Count: 1000, OPS: 172.5, Avg(us): 489, Min(us): 233, Max(us): 52991, 99th(us): 720, 99.9th(us): 928, 99.99th(us): 52991
READ_MODIFY_WRITE - Takes(s): 5.8, Count: 517, OPS: 89.3, Avg(us): 10796, Min(us): 3026, Max(us): 44415, 99th(us): 20415, 99.9th(us): 43711, 99.99th(us): 44415
UPDATE - Takes(s): 5.8, Count: 517, OPS: 89.3, Avg(us): 10344, Min(us): 2686, Max(us): 43967, 99th(us): 19967, 99.9th(us): 43295, 99.99th(us): 43967
[2023/03/28 10:44:13.720 +08:00] [INFO] [client.go:768] ["[pd] stop fetching the pending tso requests due to context canceled"] [dc-location=global]
[2023/03/28 10:44:13.721 +08:00] [INFO] [client.go:706] ["[pd] exit tso dispatcher"] [dc-location=global]
```

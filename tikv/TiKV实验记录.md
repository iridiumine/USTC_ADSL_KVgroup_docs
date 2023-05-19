# TiKV实验记录

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


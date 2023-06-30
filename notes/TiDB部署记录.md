[TOC]

# 使用TiUP部署一个本地集群实例（测试）

[使用 TiUP 部署 TiDB 集群 | PingCAP Docs](https://docs.pingcap.com/zh/tidb/stable/production-deployment-using-tiup)

TiUP：TiDB的组件管理器

安装TiUP：

```
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

## 启动集群

```
tiup playground v4.0.0 --kv 3 --pd 3 --db 3 --monitor
```

## 查看集群

```
tiup playground display
```

## 扩容集群

```
tiup playground scale-out --pd 1
```

## 缩容集群

```
tiup playground scale-in --pid <pid>
```

# 使用TiUP部署一个本地集群实例（生产）

```
tiup cluster [check|deploy|start|stop|scale-in|scale-out|...] ...
```

- check：检查环境
- deploy：根据提供的拓扑文件部署集群
- start/stop：开始/停止集群服务
- scale-out/scale-in：扩容/缩容集群

查看集群拓扑图

```
cat topology.yaml
```

## 遇到的问题

### SSH配置

[github设置添加SSH - 破男孩 - 博客园 (cnblogs.com)](https://www.cnblogs.com/ayseeing/p/3572582.html)

### SSH免密登陆

[sudo: a terminal is required to read the password； either use ……问题解决方法_akaiziyou的博客-CSDN博客](https://blog.csdn.net/akaiziyou/article/details/121678060)

### 拓扑文件

参考文档——详细最小配置模板：[docs-cn/complex-mini.yaml at master · pingcap/docs-cn · GitHub](https://github.com/pingcap/docs-cn/blob/master/config-templates/complex-mini.yaml)

- 本地主机（127.0.0.1），多个端口
- 实例数量
  - PD：1
  - KV：3（3副本保证最终一致性）
  - DB：1
- server_configs：[Search | PingCAP Docs](https://docs.pingcap.com/zh/search)

```
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/tidb-deploy"
  data_dir: "/tidb-data"

server_configs:
  pd:
    replication.location-labels: ["zone","dc","host"]     
     
     
pd_servers:
  - host: 127.0.0.1
    # ssh_port: 22
    # name: "pd-1"
    client_port: 2379
    peer_port: 2380
    # deploy_dir: "/tidb-deploy/pd-2379"
    # data_dir: "/tidb-data/pd-2379"
    # log_dir: "/tidb-deploy/pd-2379/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.pd` values.
    #   schedule.max-merge-region-size: 20
    #   schedule.max-merge-region-keys: 200000


tidb_servers:
  - host: 127.0.0.1
    # ssh_port: 22
    # port: 4000
    # status_port: 10080
    # deploy_dir: "/tidb-deploy/tidb-4000"
    # log_dir: "/tidb-deploy/tidb-4000/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.tidb` values.
    # config:
    #   log.slow-query-file: tidb-slow-overwrited.log

tikv_servers:
  - host: 127.0.0.1
    # ssh_port: 22
    port: 20160
    status_port: 20180
    deploy_dir: "/tidb-deploy/tikv-20160"
    data_dir: "/tidb-data/tikv-20160"
    log_dir: "/tidb-deploy/tikv-20160/log"
    # numa_node: "0,1"
    # # The following configs are used to overwrite the `server_configs.tikv` values.
    config:
       server.grpc-concurrency: 4
       server.labels: { zone: "zone1", dc: "dc1", host: "host1" }
  - host: 127.0.0.1
    port: 20161
    status_port: 20181
    deploy_dir: "/tidb-deploy/tikv-20161"
    data_dir: "/tidb-data/tikv-20161"
    log_dir: "/tidb-deploy/tikv-20161/log"
    config:
       server.grpc-concurrency: 4
       server.labels: { zone: "zone1", dc: "dc1", host: "host2" }
  - host: 127.0.0.1
    port: 20162
    status_port: 20182
    deploy_dir: "/tidb-deploy/tikv-20162"
    data_dir: "/tidb-data/tikv-20162"
    log_dir: "/tidb-deploy/tikv-20162/log"
    config:
       server.grpc-concurrency: 4
       server.labels: { zone: "zone1", dc: "dc1", host: "host3" }

monitoring_servers:
  - host: 127.0.0.1
    # ssh_port: 22
    # port: 9090
    # deploy_dir: "/tidb-deploy/prometheus-8249"
    # data_dir: "/tidb-data/prometheus-8249"
    # log_dir: "/tidb-deploy/prometheus-8249/log"

grafana_servers:
  - host: 127.0.0.1
    # port: 3000
    # deploy_dir: /tidb-deploy/grafana-3000

alertmanager_servers:
  - host: 127.0.0.1
    # ssh_port: 22
    # web_port: 9093
    # cluster_port: 9094
    # deploy_dir: "/tidb-deploy/alertmanager-9093"
    # data_dir: "/tidb-data/alertmanager-9093"
    # log_dir: "/tidb-deploy/alertmanager-9093/log"
```

### check不过关

可以忽略

## 指令

```
# 查看端口ssh链接报错
ssh iridiumine@127.0.0.1 -v
 
#编辑拓扑文件
gedit topology.yaml 
 
#部署
tiup cluster deploy tidb-test v5.4.0 ./topology.yaml --user iridiumine
 
# 开始启动
tiup cluster start tidb-test --init
```

## 初始化

```
Started cluster `tidb-test` successfully
The root password of TiDB database has been changed.
The new password is: 'BC2eRD!9-#0Qu$W568'.
Copy and record it to somewhere safe, it is only displayed once, and will not be stored.
The generated password can NOT be get and shown again.
```

### 密码

#### TiDB Dashboard

root

BC2eRD!9-#0Qu$W568

#### Grafana

admin

19991023zsy

### 连接TiDB数据库命令

```
mysql -u root --host 127.0.0.1 -P 4003 -p
Enter password:
BC2eRD!9-#0Qu$W568
```

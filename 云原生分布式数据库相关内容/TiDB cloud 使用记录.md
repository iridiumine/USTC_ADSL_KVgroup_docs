# 部署

## 设置

Cluster Name：Cluster0

Cloud Provider & Region：Singapore

Cluster Size

|             | Node Size (vCPU + RAM) | Node Storage | Node Quantity |
| ----------- | ---------------------- | ------------ | ------------- |
| **TiDB**    | Shared                 |              | 1             |
| **TiKV**    | Shared                 | 1 GiB        | 1             |
| **TiFlash** | Shared                 | 1 GiB        | 1             |

mysql用户名：MySQL80

mysql密码：12345678

使用默认的样例进行测试，延迟过长。

```
mysql> SELECT COUNT(*) FROM trips;
+----------+
| COUNT(*) |
+----------+
|   816090 |
+----------+
1 row in set (1.11 sec)
```

```
mysql> SELECT * FROM trips WHERE start_station_name = '8th & D St NW';
1770 rows in set (12.38 sec)(第一次测试)
1770 rows in set (1.98 sec)(第二次测试)
1770 rows in set (2.91 sec)(第三次测试)
```

## 登陆TiDB cloud

```
mysql --connect-timeout 15 -u 'Q1Ma1YUr5QHHfFo.root' -h gateway01.ap-southeast-1.prod.aws.tidbcloud.com -P 4000 -D test -p
password:19991023zsy
```

### 方向

#### ycsb

[brianfrankcooper/YCSB: Yahoo! Cloud Serving Benchmark (github.com)](https://github.com/brianfrankcooper/YCSB)

![img](https://pic1.zhimg.com/v2-b04bcf5193a302fcf3b63f446e68bffc_r.jpg)

ycsb-0.17.0版本默认不支持TiDB cloud。在开始测试之前，需要按照YCSB的扩展要求实现其对TiDB cloud的支持。

[【操作教程】利用YCSB测试巨杉数据库性能 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/23533975)

[YCSB数据库性能测试工具 - 简书 (jianshu.com)](https://www.jianshu.com/p/66937631bf0b)

[YCSB环境搭建与新数据库测试的详细指导 - 简书 (jianshu.com)](https://www.jianshu.com/p/675947cdcce4)

#### go-ycsb



#### 其他云原生数据库测试工具



#### 关于TiDB的测试

https://tidb.net/blog/b8dccf46

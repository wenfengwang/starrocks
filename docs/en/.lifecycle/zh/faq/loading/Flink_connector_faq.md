---
displayed_sidebar: English
---

# Flink 连接器

## flink-connector-jdbc_2.11 sink 在 StarRocks 中延迟了 8 小时

**问题描述：**

在 Flink 中，localtimestap 函数生成的时间是正常的。但是当数据下沉到 StarRocks 时，时间变成了晚 8 个小时。Flink 服务器和 StarRocks 服务器位于同一时区，即亚洲/上海 UTC/GMT+08:00。Flink 版本为 1.12。驱动程序：flink-connector-jdbc_2.11。请问如何解决这个问题？

**解决方案:**

请尝试在 Flink sink 表中配置时间参数 'server-time-zone' = 'Asia/Shanghai'。您也可以在 JDBC URL 中添加 &serverTimezone=Asia/Shanghai。以下是一个示例：

```sql
CREATE TABLE sk (
    sid INT,
    local_dtm TIMESTAMP,
    curr_dtm TIMESTAMP
)
WITH (
    'connector' = 'jdbc',
    'url' = 'jdbc:mysql://192.168.110.66:9030/sys_device?characterEncoding=utf-8&serverTimezone=Asia/Shanghai',
    'table-name' = 'sink',
    'driver' = 'com.mysql.jdbc.Driver',
    'username' = 'sr',
    'password' = 'sr123',
    'server-time-zone' = 'Asia/Shanghai'
);
```

## 在 flink 导入时，只有部署在 StarRocks 集群中的 Kafka 集群才能被导入

**问题描述：**

```SQL
failed to query watermark offset, err: Local: Bad message format
```

**解决方案:**

Kafka 通信需要主机名。用户需要在 StarRocks 集群节点中配置主机名解析 /etc/hosts。

## StarRocks 能批量导出 'create table statements' 吗？

**解决方案:**

您可以使用 StarRocks Tools 来导出这些语句。

## BE 请求的内存没有被释放回操作系统

这是一个正常现象，因为操作系统分配给数据库的大块内存在分配时会被保留，并在释放时延迟，以便重用内存，使内存分配更加方便。建议用户通过长时间监控内存使用情况来验证测试环境，以确认内存是否能够被释放。

## 下载后 Flink connector 无法工作

**问题描述：**

该包需要通过阿里云镜像地址获得。

**解决方案:**

请确保 `/etc/maven/settings.xml` 中的镜像部分全部配置为通过阿里云镜像地址获取。

如果已配置，则将其更改为以下内容：

 <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun public repo</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>


## Flink-connector-StarRocks 中参数 sink.buffer-flush.interval-ms 的含义

**问题描述：**

```plain
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | 刷新时间间隔，              |
|  interval-ms         |          |            |        | 范围：[1000ms, 3600000ms]   |
+----------------------+--------------------------------------------------------------+
```

如果这个参数设置为 15 秒，而检查点间隔设置为 5 分钟，这个值还会生效吗？

**解决方案:**

无论三个阈值中的哪一个先达到，先达到的那个阈值就会生效。这不受检查点间隔值的影响，检查点间隔值只对 exactly_once 生效。at_least_once 使用 interval-ms。
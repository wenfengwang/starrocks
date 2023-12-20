---
displayed_sidebar: English
---

# Flink 连接器

## flink-connector-jdbc_2.11 sink 在 StarRocks 中延迟了 8 小时

**问题描述：**

在 Flink 中，localtimestap 函数生成的时间是正常的。但数据沉入 StarRocks 后，时间晚了 8 个小时。Flink 服务器和 StarRocks 服务器位于同一时区，即亚洲/上海 UTC/GMT+08:00。Flink 版本为 1.12，使用的驱动程序是 flink-connector-jdbc_2.11。请问如何解决这个问题？

**解决方案:**

请尝试在 Flink Sink 表中配置时间参数 'server-time-zone' = 'Asia/Shanghai'。您还可以在 JDBC URL 中添加 &serverTimezone=Asia/Shanghai。以下是一个示例：

```sql
CREATE TABLE sk (
    sid int,
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

## 在 Flink 导入时，只有部署在 StarRocks 集群中的 Kafka 集群才能被导入

**问题描述：**

```SQL
failed to query wartermark offset, err: Local: Bad message format
```

**解决方案:**

Kafka 通信需要主机名。用户需要在 StarRocks 集群节点的 /etc/hosts 文件中配置主机名解析。

## StarRocks 能批量导出“创建表”的语句吗？

**解决方案:**

您可以使用 StarRocks 工具来批量导出这些语句。

## BE 请求的内存未被释放回操作系统

这是一种正常现象，因为从操作系统分配给数据库的大块内存在分配时会被保留，在释放时会延迟回收，以便重复使用内存，使内存分配更加便捷。建议用户通过长期监控内存使用情况来验证测试环境，以判断内存是否可以被释放。

## 下载后的 Flink 连接器无法使用

**问题描述：**

需要通过阿里云镜像地址来获取该软件包。

**解决方案:**

请确保 /etc/maven/settings.xml 中的镜像部分都配置为通过阿里云镜像地址来获取。

如果已经配置，请将其更改为以下内容：

 <mirror>
    <id>aliyunmaven </id>
    <mirrorf>central</mirrorf>
    <name>aliyun public repo</name>
    <url>https: //maven.aliyun.com/repository/public</url>
</mirror>


## 在 Flink-connector-StarRocks 中参数 sink.buffer-flush.interval-ms 的含义

**问题描述：**

```plain
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

如果将该参数设置为 15 秒，而检查点间隔设置为 5 分钟，这个参数设置是否仍然有效？

**解决方案:**

三个阈值中任何一个首先达到的都会首先生效。这不受检查点间隔值的影响，检查点间隔值只对 exactly_once 生效。参数 interval-ms 用于 at_least_once 策略。

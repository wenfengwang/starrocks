---
displayed_sidebar: English
---

# Flink 连接器

## flink-connector-jdbc_2.11sink 在 StarRocks 中晚了 8 小时

**问题描述：**

localtimestap 函数在 Flink 中生成的时间是正常的。但是当它沉入 StarRocks 时，会晚8个小时。Flink 服务器和 StarRocks 服务器位于同一时区，即亚洲/上海 UTC/GMT+08:00。Flink 版本为 1.12。驱动程序：flink-connector-jdbc_2.11。我可以问一下如何解决这个问题吗？

**解决方案：**

请尝试在 Flink sink 表中配置时间参数 'server-time-zone' = 'Asia/Shanghai'。您也可以在 jdbc url 中添加 &serverTimezone=Asia/Shanghai。示例如下：

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

## 在 Flink 导入中，只能导入部署在 StarRocks 集群中的 kafka 集群

**问题描述：**

```SQL
failed to query wartermark offset, err: Local: Bad message format
```

**解决方案：**

Kafka 通信需要主机名。用户需要在 StarRocks 集群节点上配置主机名解析 /etc/hosts。

## StarRocks 可以批量导出 'create table 语句' 吗？

**解决方案：**

您可以使用 StarRocks 工具来导出这些语句。

## BE 请求的内存没有释放回操作系统

这是正常现象，因为从操作系统分配给数据库的大块内存在分配期间被保留，并在释放期间被延迟，以便重用内存并使内存分配更加方便。建议用户通过监视较长时间内的内存使用情况来验证测试环境，以查看是否可以释放内存。

## Flink 连接器在下载后无法正常工作

**问题描述：**

此软件包需要通过阿里云镜像地址获取。

**解决方案：**

请确保 `/etc/maven/settings.xml` 文件中的镜像部分都配置为通过阿里云镜像地址获取。

如果已经配置，请将其更改为以下内容：

 <mirror>
    <id>aliyunmaven </id>
    <mirrorf>central</mirrorf>
    <name>aliyun public repo</name>
    <url>https: //maven.aliyun.com/repository/public</url>
</mirror>

## Flink-connector-StarRocks 中参数 sink.buffer-flush.interval-ms 的含义

**问题描述：**

```plain text
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

如果该参数设置为 15s，checkpoint 间隔等于 5 mins，该值是否仍然有效？

**解决方案：**

无论哪个阈值先达到，都将首先生效。这不受检查点间隔值的影响，检查点间隔值仅适用于一次。Interval-ms 由 at_least_once 使用。

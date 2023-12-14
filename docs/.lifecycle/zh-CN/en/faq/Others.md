---
displayed_sidebar: "Chinese"
---

# 常见问题

本主题提供了一些常见问题的解答。

## VARCHAR（32）和STRING占用相同的存储空间吗？

两者都是可变长度数据类型。当存储相同长度的数据时，VARCHAR（32）和STRING占用相同的存储空间。

## VARCHAR（32）和STRING执行数据查询时是否一样？

是的。

## 我将从Oracle导入的TXT文件的字符集设置为UTF-8后，为什么仍然显示乱码？

要解决此问题，请按照以下步骤执行：

1. 例如，有一个名为 **original** 的文件，其文本是乱码。该文件的字符集为ISO-8859-1。运行以下代码以获取文件的字符集。

    ```plaintext
    file --mime-encoding origin.txt
    origin.txt: iso-8859-1
    ```

2. 运行 `iconv` 命令将该文件的字符集转换为UTF-8。

    ```plaintext
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. 转换后，该文件的文本仍然显示乱码。然后，可以将该文件的字符集重新设置为GBK，并再次转换字符集为UTF-8。

    ```plaintext
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## MySQL中STRING的长度定义与StarRocks中定义的长度相同吗？

对于VARCHAR（n），StarRocks按字节定义“n”，而MySQL按字符定义“n”。根据UTF-8，一个中文字符等于三个字节。当StarRocks和MySQL将“n”定义为相同数字时，MySQL保存的字符数是StarRocks的三倍。

## 表的分区字段的数据类型可以是FLOAT、DOUBLE或DECIMAL吗？

不可以，仅支持DATE、DATETIME和INT。

## 如何查看表中数据所占用的存储空间？

执行SHOW DATA语句来查看相应的存储空间。您还可以查看数据量、副本数以及行数。

**注意**：数据统计存在时间延迟。

## 如何申请StarRocks数据库的配额增加？

要申请配额增加，请运行以下代码：

```plaintext
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocks是否支持通过执行UPSERT语句来更新表中特定字段？

StarRocks 2.2及更高版本支持使用主键表来更新表中特定字段。StarRocks 1.9及更高版本支持使用主键表来更新表中所有字段。有关更多信息，请参见StarRocks 2.2中的[主键表](../table_design/table_types/primary_key_table.md)。

## 如何在两个表或两个分区之间交换数据？

执行SWAP WITH语句来在两个表或两个分区之间交换数据。SWAP WITH语句比INSERT OVERWRITE语句更安全。在交换数据之前，首先检查数据，然后查看交换后的数据是否与交换前的数据一致。

- 交换两个表：例如，有一个名为table1的表。如果要用另一个表替换table1，请执行以下步骤：

    1. 创建一个名为table2的新表。

        ```SQL
        create table2 like table1;
        ```

    2. 使用Stream Load、Broker Load或Insert Into从table1加载数据到table2。

    3. 用table2替换table1。

        ```SQL
        ALTER TABLE table1 SWAP WITH table2;
        ```

        这样做可以准确地将数据加载到table1中。

- 交换两个分区：例如，有一个名为table1的表。如果要替换table1中的分区数据，请执行以下步骤：

    1. 创建一个临时分区。

        ```SQL
        ALTER TABLE table1

        ADD TEMPORARY PARTITION tp1

        VALUES LESS THAN("2020-02-01");
        ```

    2. 将table1的分区数据加载到临时分区中。

    3. 用临时分区替换table1的分区。

        ```SQL
        ALTER TABLE table1

        REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
        ```

## 重新启动前端（FE）时出现错误“error to open replicated environment, will exit”

此错误是由于BDBJE的bug导致的。要解决此问题，请将BDBJE版本更新为1.17或更高版本。

## 查询新的Apache Hive表时出现错误“Broker list path exception”

### 问题描述

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解决方法

联系StarRocks技术支持，检查namenode的地址和端口是否正确，以及您是否有权限访问namenode的地址和端口。

## 查询新的Apache Hive表时出现错误“get hive partition metadata failed”

### 问题描述

```plaintext
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解决方法

确保网络已连接，并将**host**文件上传到StarRocks集群中的每个后端（BE）。

## 在Apache Hive中访问ORC外部表时出现错误“do_open failed. reason = Invalid ORC postscript length”

### 问题描述

Apache Hive的元数据被缓存在FE中。但是在StarRocks更新元数据时存在两小时的时间差。在StarRocks完成更新之前，如果您在Apache Hive表中插入新数据或更新数据，则BE扫描的HDFS中的数据和FE获取的数据将不同。因此，会出现此错误。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解决方法

要解决此问题，请执行以下操作之一：

- 将当前版本升级为StarRocks 2.2或更高版本。
- 手动刷新Apache Hive表。有关更多信息，请参见[元数据缓存策略](../data_source/External_table.md)。

## 连接MySQL的外部表时出现错误“caching_sha2_password cannot be loaded”

### 问题描述

MySQL 8.0的默认认证插件是caching_sha2_password。MySQL 5.7的默认认证插件是mysql_native_password。由于使用了错误的认证插件，因此会出现此错误。

### 解决方法

要解决此问题，请执行以下操作之一：

- 连接到StarRocks。

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

- 修改`my.cnf`文件。

```plaintext
vim my.cnf

[mysqld]

default_authentication_plugin=mysql_native_password
```

## 删除表后立即释放磁盘空间？

如果执行DROP TABLE语句删除表，StarRocks需要一段时间来释放分配的磁盘空间。要立即释放分配的磁盘空间，请执行DROP TABLE FORCE语句删除表。执行DROP TABLE FORCE语句时，StarRocks将直接删除表，而无需检查其中是否有未完成的事件。我们建议您谨慎执行DROP TABLE FORCE语句，因为一旦删除表，将无法恢复。

## 如何查看当前的StarRocks版本？

运行 `select current_version();` 命令或CLI命令 `./bin/show_fe_version.sh` 来查看当前版本。

## 如何设置FE的内存大小？

FE使用的内存存储了元数据。您可以根据下表中的分片数设置FE的内存大小。例如，如果分片数低于100万，应为FE分配至少16 GB的内存。您可以在**fe.conf**文件的**JAVA_OPTS**配置项中配置参数`-Xms` 和 `-Xmx` 的值，并且参数`-Xms` 和 `-Xmx` 的值应该一致。请注意，由于任何FE都可以被选举为Leader，因此配置应在所有FE上保持一致。

| 分片数      | 每个FE的内存大小 |
| ---------- | ----------- |
| 低于100万 | 16 GB       |
| 100万～200万 | 32 GB       |
| 200万～500万 | 64 GB       |
| 500万～1000万   | 128 GB      |

## StarRocks如何计算查询时间？

StarRocks支持使用多个线程查询数据。查询时间指的是多个线程查询数据所用的时间。

## StarRocks是否支持在本地导出数据时设置路径？

不支持。

## StarRocks的并发上限是多少？

您可以根据实际业务场景或模拟业务场景进行并发限制测试。根据一些用户的反馈，最大可以达到20000QPS或30000QPS。

## 为什么StarRocks首次进行SSB测试的性能要比第二次慢？

首次查询的磁盘读取速度与磁盘的性能有关。在第一次查询后，为后续查询生成了页面缓存，所以查询比之前更快。

## 集群至少需要配置多少个BE？

StarRocks支持单节点部署，因此至少需要配置一个BE。BE需要在AVX2上运行，因此建议您在配置至少8核和16GB或更高配置的机器上部署BE。

## 在使用Apache Superset对StarRocks中的数据进行可视化时，如何设置数据权限？

您可以创建一个新的用户账号，然后通过对用户授予表查询权限来设置数据权限。

## 我将`enable_profile`设置为`true`后，为什么概要显示失败了？

报告仅提交给主FE以供访问。

## 如何检查StarRocks表中的字段注释？

运行`show create table xxx`命令。

## 当我创建一个表时，如何指定NOW()函数的默认值？

只有StarRocks 2.1或更新版本支持为函数指定默认值。对于早于StarRocks 2.1的版本，您只能为函数指定一个常量。

## 如何释放BE节点的存储空间？

您可以使用`rm -rf`命令删除`trash`目录。如果您已经从快照中恢复了数据，可以删除`snapshot`目录。

## 可以为BE节点添加额外的磁盘吗？

可以。您可以将磁盘添加到BE配置项`storage_root_path`指定的目录。
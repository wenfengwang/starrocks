---
displayed_sidebar: English
---

# 其他常见问题

本主题提供了一些常见问题的答案。

## VARCHAR（32）和STRING是否占用相同的存储空间？

两者都是可变长度数据类型。当您存储相同长度的数据时，VARCHAR（32）和STRING占用相同的存储空间。

## VARCHAR（32）和STRING在数据查询时执行相同的操作吗？

是的。

## 为什么将从Oracle导入的TXT文件的字符集设置为UTF-8后，仍然会出现乱码？

要解决这个问题，请执行以下步骤：

1. 例如，有一个名为**original**的文件，其文本是乱码。该文件的字符集为ISO-8859-1。运行以下代码获取文件的字符集。

    ```plaintext
    file --mime-encoding origin.txt
    origin.txt: iso-8859-1
    ```

2. 运行`iconv`命令将该文件的字符集转换为UTF-8格式。

    ```plaintext
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. 转换后，该文件的文本仍然显示为乱码。然后，您可以重新将该文件的字符集转换为GBK，并再次将字符集转换为UTF-8。

    ```plaintext
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## MySQL定义的STRING长度和StarRocks定义的长度一样吗？

对于VARCHAR（n），StarRocks以字节定义“n”，而MySQL以字符定义“n”。根据UTF-8，一个汉字等于三个字节。当StarRocks和MySQL将“n”定义为相同的数字时，MySQL保存的字符数是StarRocks的3倍。

## 表的分区字段的数据类型可以是FLOAT、DOUBLE或DECIMAL吗？

不可以，仅支持DATE、DATETIME和INT。

## 如何查看表中数据占用的存储空间？

执行SHOW DATA语句查看对应的存储空间。您还可以查看数据量、副本数和行数。

**注意**：数据统计存在时间延迟。

## 如何请求增加StarRocks数据库的配额？

要请求增加配额，请运行以下代码：

```plaintext
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocks是否支持通过执行UPSERT语句来更新表中的特定字段？

StarRocks 2.2及更高版本支持使用主键表更新表中的特定字段。StarRocks 1.9及更高版本支持使用主键表更新表中的所有字段。有关更多信息，请参见[主键表](../table_design/table_types/primary_key_table.md)中的StarRocks 2.2。

## 如何在两个表或两个分区之间交换数据？

执行SWAP WITH语句，在两个表或两个分区之间交换数据。SWAP WITH语句比INSERT OVERWRITE语句更安全。在交换数据之前，先检查数据，然后查看交换后的数据是否与交换前的数据一致。

- 交换两个表：例如，有一个名为table 1的表。如果要用另一个表替换table 1，请执行以下步骤：

    1. 创建一个名为table 2的新表。

        ```SQL
        create table2 like table1;
        ```

    2. 使用Stream Load、Broker Load或Insert Into将数据从table 1加载到table 2中。

    3. 用table 2替换table 1。

        ```SQL
        ALTER TABLE table1 SWAP WITH table2;
        ```

        这样，数据就准确地加载到table 1中。

- 交换两个分区：例如，有一个名为table 1的表。如果要替换table 1中的分区数据，请执行以下步骤：

    1. 创建一个临时分区。

        ```SQL
        ALTER TABLE table1

        ADD TEMPORARY PARTITION tp1

        VALUES LESS THAN("2020-02-01");
        ```

    2. 将table 1的分区数据加载到临时分区中。

    3. 用临时分区替换table 1的分区。

        ```SQL
        ALTER TABLE table1

        REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
        ```

## 当我重新启动前端（FE）时出现错误“打开复制环境时出错，将退出”

这个错误是由BDBJE的bug引起的。要解决这个问题，请将BDBJE版本更新到1.17或更高版本。

## 当我从新的Apache Hive表中查询数据时，会出现错误“代理列表路径异常”

### 问题描述

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解决方案

联系StarRocks技术支持，检查namenode的地址和端口是否正确，以及您是否有权访问namenode的地址和端口。

## 当我从新的Apache Hive表中查询数据时，会出现错误“获取hive分区元数据失败”

### 问题描述

```plaintext
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解决方案

确保网络已连接，并将**host**文件上传到StarRocks集群中的每个后端（BE）。

## 当我在Apache Hive中访问ORC外部表时，出现错误“do_open失败。reason =无效的ORC后记长度”

Apache Hive的元数据被缓存在FE中。但是，StarRocks更新元数据需要两个小时的时间差。在StarRocks完成更新之前，如果在Apache Hive表中插入新数据或更新数据，则BE扫描的HDFS数据和FE获取的数据是不同的。因此，会出现这个错误。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解决方案

要解决这个问题，请执行以下操作之一：

- 将当前版本升级至StarRocks 2.2或更高版本。
- 手动刷新Apache Hive表。有关详细信息，请参见[元数据缓存策略](../data_source/External_table.md)。

## 当我连接MySQL的外部表时，出现错误“无法加载caching_sha2_password”

### 问题描述

MySQL 8.0的默认身份验证插件是caching_sha2_password。MySQL 5.7的默认身份验证插件是mysql_native_password。出现这个错误是因为您使用了错误的身份验证插件。

### 解决方案

要解决这个问题，请执行以下操作之一：

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

## 删除表后如何立即释放磁盘空间？

如果执行DROP TABLE语句删除表，StarRocks需要一段时间才能释放分配的磁盘空间。要立即释放已分配的磁盘空间，请执行DROP TABLE FORCE语句删除表。当您执行DROP TABLE FORCE语句时，StarRocks会直接删除该表，而不会检查其中是否有未完成的事件。我们建议您谨慎执行DROP TABLE FORCE语句，因为一旦删除了表，就无法恢复它。

## 如何查看当前版本的StarRocks？

运行`select current_version();`命令或CLI命令`./bin/show_fe_version.sh`，查看当前版本。

## 如何设置FE的内存大小？

元数据存储在FE使用的内存中。您可以根据平板电脑的数量设置FE的内存大小，如下表所示。例如，如果平板电脑数量低于100万，则应为FE分配至少16GB的内存。您可以在**fe.conf**文件的**JAVA_OPTS**配置项中配置参数**-Xms**和**-Xmx**的值，这两个参数的值应保持一致。请注意，所有FE的配置都应该相同，因为任何FE都可以被选为Leader。

| 平板电脑数量    | 每个FE的内存大小 |
| -------------- | ----------- |
| 100万以下      | 16GB        |
| 1~200万 | 32GB        |
| 2~500万 | 64GB        |
| 5~1000万   | 128GB       |

## StarRocks是如何计算查询时间的？

StarRocks支持使用多线程查询数据。查询时间是指多个线程查询数据所花费的时间。

## StarRocks是否支持在本地导出数据时设置路径？

不支持。

## StarRocks的并发上限是多少？

您可以根据实际业务场景或模拟业务场景测试并发限制。根据部分用户反馈，最高可达20000 QPS或30000 QPS。

## 为什么StarRocks第一次SSB测试的性能比第二次慢？

第一个查询的磁盘读取速度与磁盘性能有关。第一次查询后，会为后续查询生成页面缓存，因此查询速度比以前快。

## 一个集群至少需要配置多少个BE？

StarRocks支持单节点部署，因此至少需要配置一个BE。BE需要与AVX2一起运行，因此建议您在具有8核和16GB或更高配置的计算机上部署BE。

## 使用Apache Superset在StarRocks中可视化数据时，如何设置数据权限？

您可以创建一个新的用户帐户，然后通过向用户授予表查询权限来设置数据权限。

## 为什么配置文件在我设置为后无法显示`enable_profile` `true`？

报告仅提交给Leader FE进行访问。

## 如何查看StarRocks表格中的字段注解？

运行`show create table xxx`命令。

## 创建表时，如何指定NOW（）函数的默认值？

仅StarRocks 2.1或更高版本支持指定函数默认值。StarRocks 2.1之前的版本只能为函数指定一个常量。

## 如何释放BE节点的存储空间？

您可以使用`rm -rf`命令删除`trash`目录。如果您已经从快照中恢复了数据，则可以删除`snapshot`目录。

## 是否可以向BE节点添加额外的磁盘？

可以。您可以将磁盘添加到BE配置项指定的目录下`storage_root_path`。

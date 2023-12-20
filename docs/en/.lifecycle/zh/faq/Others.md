---
displayed_sidebar: English
---

# 其他常见问题解答

本主题提供了一些常见问题的答案。

## VARCHAR(32)和STRING占用相同的存储空间吗？

两者都是可变长度数据类型。当存储相同长度的数据时，VARCHAR(32)和STRING占用相同的存储空间。

## VARCHAR(32)和STRING在数据查询中的性能是否相同？

是的。

## 为什么从Oracle导入的TXT文件即使设置字符集为UTF-8后仍然出现乱码？

要解决此问题，请执行以下步骤：

1. 例如，存在一个名为**original**的文件，其文本显示为乱码。该文件的字符集是ISO-8859-1。运行以下代码以获取文件的字符集。

   ```plaintext
   file --mime-encoding origin.txt
   origin.txt: iso-8859-1
   ```

2. 运行`iconv`命令将该文件的字符集转换为UTF-8。

   ```plaintext
   iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
   ```

3. 转换后，该文件的文本仍然显示为乱码。此时，您可以再次将文件的字符集从GBK转换为UTF-8。

   ```plaintext
   iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
   ```

## MySQL定义的STRING长度和StarRocks定义的长度是否相同？

对于VARCHAR(n)，StarRocks按字节定义“n”，而MySQL按字符定义“n”。根据UTF-8编码，一个汉字等于三个字节。因此，当StarRocks和MySQL将“n”定义为相同数值时，MySQL能存储的汉字数量是StarRocks的三倍。

## 表的分区字段的数据类型可以是FLOAT、DOUBLE或DECIMAL吗？

不可以，只支持DATE、DATETIME和INT类型。

## 如何查看表中数据占用的存储空间？

执行SHOW DATA语句可以查看对应的存储空间。您还可以查看数据量、副本数和行数。

**注意**：数据统计存在时间延迟。

## 如何为StarRocks数据库请求配额增加？

要请求增加配额，请运行以下代码：

```plaintext
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocks是否支持通过执行UPSERT语句更新表中的特定字段？

StarRocks 2.2及更高版本支持使用主键表更新表中的特定字段。StarRocks 1.9及更高版本支持使用主键表更新表中的所有字段。更多信息请参见[主键表](../table_design/table_types/primary_key_table.md)文档。

## 如何在两个表或两个分区之间交换数据？

执行SWAP WITH语句可以在两个表或两个分区之间交换数据。SWAP WITH语句比INSERT OVERWRITE语句更安全。在交换数据之前，请先检查数据，然后确认交换后的数据与交换前是否一致。

- 交换两个表：例如，有一个名为table1的表，如果您想用另一个表替换table1，请执行以下步骤：

1.     创建一个新表，命名为table2。

       ```SQL
       CREATE TABLE table2 LIKE table1;
       ```

2.     使用Stream Load、Broker Load或Insert Into将数据从table1加载到table2。

3.     用table2替换table1。

       ```SQL
       ALTER TABLE table1 SWAP WITH table2;
       ```

       这样，数据就准确地加载到了table1。

- 交换两个分区：例如，有一个名为table1的表，如果您想替换table1中的分区数据，请执行以下步骤：

1.     创建一个临时分区。

       ```SQL
       ALTER TABLE table1
       
       ADD TEMPORARY PARTITION tp1
       
       VALUES LESS THAN ("2020-02-01");
       ```

2.     将table1中的分区数据加载到临时分区。

3.     用临时分区替换table1的分区。

       ```SQL
       ALTER TABLE table1
       
       REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
       ```

## 当我重启前端（FE）时，出现错误“打开复制环境失败，将退出”

这个错误是由于BDBJE的bug引起的。要解决此问题，请将BDBJE版本升级到1.17或更高。

## 当我从新的Apache Hive表查询数据时，出现错误“Broker list path exception”

### 问题描述

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解决方案

联系StarRocks技术支持，检查namenode的地址和端口是否正确，以及您是否有权限访问namenode的地址和端口。

## 当我从新的Apache Hive表查询数据时，出现错误“获取Hive分区元数据失败”

### 问题描述

```plaintext
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解决方案

确保网络已连接，并将**hosts**文件上传到StarRocks集群中的每个后端（BE）。

## 当我访问Apache Hive中的ORC外部表时，出现错误“do_open failed. reason = Invalid ORC postscript length”

### 问题描述

Apache Hive的元数据缓存在FE中。但StarRocks更新元数据有两小时的延迟。在StarRocks完成更新之前，如果在Apache Hive表中插入新数据或更新数据，则BE扫描到的HDFS中的数据与FE获取的数据不一致。因此，会出现此错误。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解决方案

要解决此问题，请执行以下操作之一：

- 将当前版本升级到StarRocks 2.2或更高版本。
- 手动刷新Apache Hive表的元数据。更多信息请参见[元数据缓存策略](../data_source/External_table.md)。

## 连接MySQL外部表时出现错误“caching_sha2_password无法加载”

### 问题描述

MySQL 8.0的默认身份验证插件是caching_sha2_password。MySQL 5.7的默认身份验证插件是mysql_native_password。出现此错误是因为使用了错误的身份验证插件。

### 解决方案

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

## 删除表后如何立即释放磁盘空间？

如果执行DROP TABLE语句删除表，StarRocks需要一段时间来释放分配的磁盘空间。要立即释放分配的磁盘空间，请执行DROP TABLE FORCE语句删除表。当执行DROP TABLE FORCE语句时，StarRocks会直接删除表，而不检查表中是否有未完成的事件。我们建议您谨慎使用DROP TABLE FORCE语句。因为一旦表被删除，就无法恢复。

## 如何查看StarRocks的当前版本？

运行`SELECT CURRENT_VERSION();`命令或CLI命令`./bin/show_fe_version.sh`来查看当前版本。

## 如何设置FE的内存大小？

元数据存储在FE使用的内存中。您可以根据平板电脑的数量来设置FE的内存大小，如下表所示。例如，如果平板电脑的数量低于100万，您应该为FE分配至少16GB的内存。您可以在`fe.conf`文件中的`JAVA_OPTS`配置项中设置参数`-Xms`和`-Xmx`的值，且`-Xms`和`-Xmx`的值应该一致。注意，所有FE的配置应该相同，因为任何FE都可能被选为Leader。

| 平板电脑数量 | 每个FE的内存大小 |
| --- | --- |
| 100万以下 | 16 GB |
| 100万到200万 | 32 GB |
| 200万到500万 | 64 GB |
| 500万到1000万 | 128 GB |

## StarRocks如何计算其查询时间？

StarRocks支持使用多线程查询数据。查询时间指的是多个线程查询数据所用的时间。

## StarRocks支持在本地导出数据时设置路径吗？

不支持。

## StarRocks的并发上限是多少？

您可以根据实际业务场景或模拟业务场景来测试并发限制。根据一些用户的反馈，最高可以达到20000 QPS或30000 QPS。

## 为什么StarRocks第一次SSB测试的性能比第二次慢？

首次查询读取磁盘的速度取决于磁盘的性能。第一次查询后，为后续查询生成了页面缓存，因此查询速度比之前更快。

## 一个集群至少需要配置多少个BE？

StarRocks支持单节点部署，因此您至少需要配置一个BE。BE需要运行AVX2，因此我们建议您在至少具备8核和16GB配置的机器上部署BE。

## 使用Apache Superset可视化StarRocks中的数据时，如何设置数据权限？

您可以创建一个新的用户账户，然后通过授予该用户对表查询的权限来设置数据权限。

## 为什么在`enable_profile`设置为`true`后，profile无法显示？

报告只提交给Leader FE进行访问。

## 如何查看StarRocks表中的字段注释？

运行`SHOW CREATE TABLE xxx`命令。

## 创建表时，如何指定NOW()函数的默认值？

只有StarRocks 2.1或更高版本支持为函数指定默认值。对于StarRocks 2.1之前的版本，您只能为函数指定常量。

## 如何释放BE节点的存储空间？

您可以使用`rm -rf`命令删除`trash`目录。如果您已经从快照中恢复了数据，则可以删除`snapshot`目录。

## BE节点可以添加额外的磁盘吗？

可以。您可以将磁盘添加到BE配置项`storage_root_path`指定的目录中。
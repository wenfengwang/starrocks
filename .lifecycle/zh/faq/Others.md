---
displayed_sidebar: English
---

# 其他常见问题解答

本主题提供了一些常见问题的答案。

## VARCHAR(32)和STRING占用相同的存储空间吗？

两者都是可变长度的数据类型。当存储相同长度的数据时，VARCHAR(32)和STRING占用的存储空间是相同的。

## VARCHAR(32)和STRING在数据查询中的性能是否相同？

是的。

## 为什么从Oracle导入的TXT文件，在我将字符集设置为UTF-8后，仍然显示乱码？

要解决此问题，请按照以下步骤操作：

1. 比如，有一个文件名为**original**的文本文件出现了乱码，该文件的字符集是ISO-8859-1。运行以下代码以获取文件的字符集。

   ```plaintext
   file --mime-encoding origin.txt
   origin.txt: iso-8859-1
   ```

2. 运行iconv命令将文件的字符集转换为UTF-8。

   ```plaintext
   iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
   ```

3. 转换后，文件文本仍显示乱码。这时，您可以尝试将文件的字符集重新设为GBK，然后再次转换为UTF-8。

   ```plaintext
   iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
   ```

## MySQL定义的STRING长度和StarRocks定义的长度是否相同？

对于VARCHAR(n)，StarRocks是按字节定义“n”的，而MySQL是按字符定义“n”的。根据UTF-8编码，一个汉字相当于三个字节。因此，当StarRocks和MySQL将“n”定义为相同数值时，MySQL能存储的汉字数量是StarRocks的三倍。

## 表的分区字段的数据类型可以是FLOAT、DOUBLE或DECIMAL吗？

不可以，只支持DATE、DATETIME和INT类型。

## 如何查看表中数据所占用的存储空间？

执行SHOW DATA语句可以查看相应的存储空间。您还可以查看数据量、副本数和行数。

**注意**：数据统计存在时间延迟。

## 如何申请StarRocks数据库的配额增加？

要申请增加配额，请执行以下代码：

```plaintext
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocks是否支持通过执行UPSERT语句来更新表中的特定字段？

StarRocks 2.2及更高版本支持使用主键表来更新表中的特定字段。StarRocks 1.9及更高版本支持使用主键表来更新表中的所有字段。更多信息请参见[主键表](../table_design/table_types/primary_key_table.md)在StarRocks 2.2中。

## 如何在两个表或两个分区之间交换数据？

执行SWAP WITH语句可以在两个表或两个分区之间交换数据。SWAP WITH语句比INSERT OVERWRITE语句更安全。在交换数据之前，请先检查数据，然后确认交换后的数据是否与交换前的数据一致。

- 交换两个表：例如，有一个名为table1的表，如果您想用另一个表替换table1，请按以下步骤操作：

1.     创建一个新表，命名为table2。

       ```SQL
       create table2 like table1;
       ```

2.     使用Stream Load、Broker Load或Insert Into将table1中的数据加载到table2中。

3.     用table2替换table1。

       ```SQL
       ALTER TABLE table1 SWAP WITH table2;
       ```

       这样做可以确保数据准确地加载到table1中。

- 交换两个分区：例如，有一个名为table1的表，如果您想替换table1中的分区数据，请按以下步骤操作：

1.     创建一个临时分区。

       ```SQL
       ALTER TABLE table1
       
       ADD TEMPORARY PARTITION tp1
       
       VALUES LESS THAN("2020-02-01");
       ```

2.     将table1中的分区数据加载到临时分区中。

3.     用临时分区替换table1的分区。

       ```SQL
       ALTER TABLE table1
       
       REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
       ```

## 当我重启前端（FE）时，出现了“打开复制环境时出错，将退出”的错误

这个错误是由于BDBJE的bug引起的。要解决这个问题，请将BDBJE版本升级到1.17或更高版本。

## 当我从新的Apache Hive表中查询数据时，出现了“Broker list path exception”的错误

### 问题描述

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解决方案

联系StarRocks技术支持，核实namenode的地址和端口是否正确，以及您是否有权限访问namenode的地址和端口。

## 当我从新的Apache Hive表中查询数据时，出现了“获取Hive分区元数据失败”的错误

### 问题描述

```plaintext
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解决方案

确保网络已连接，并将**主机**文件上传到您的StarRocks集群中的每个后端（BE）。

## 当我访问Apache Hive中的ORC外部表时，出现了“do_open failed.reason = Invalid ORC postscript length”的错误

### 问题描述

Apache Hive的元数据被缓存在FE中。但StarRocks更新元数据存在两小时的延时。在StarRocks完成更新之前，如果您在Apache Hive表中插入新数据或更新数据，BE扫描到的HDFS中的数据与FE获取的数据会不同，因此会出现这个错误。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解决方案

要解决这个问题，您可以执行以下操作之一：

- 将您当前的版本升级到StarRocks 2.2或更高版本。
- 手动刷新您的Apache Hive表。更多信息请参见[元数据缓存策略](../data_source/External_table.md)。

## 连接MySQL外部表时出现“caching_sha2_password无法加载”的错误

### 问题描述

MySQL 8.0的默认认证插件是caching_sha2_password。MySQL 5.7的默认认证插件是mysql_native_password。出现这个错误是因为您使用了不正确的认证插件。

### 解决方案

要解决这个问题，请执行以下操作之一：

- 连接到StarRocks。

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

- 修改my.cnf文件。

```plaintext
vim my.cnf

[mysqld]

default_authentication_plugin=mysql_native_password
```

## 删除表后如何立即释放磁盘空间？

如果您执行DROP TABLE语句删除一个表，StarRocks会在一段时间后释放分配的磁盘空间。要立即释放分配的磁盘空间，请执行DROP TABLE FORCE语句删除表。当您执行DROP TABLE FORCE语句时，StarRocks会直接删除表，而不检查表中是否有未完成的事件。我们建议您谨慎使用DROP TABLE FORCE语句。因为一旦表被删除，您将无法恢复它。

## 如何查看StarRocks的当前版本？

运行select current_version();命令或CLI命令./bin/show_fe_version.sh来查看当前版本。

## 如何设置FE的内存大小？

元数据存储在FE使用的内存中。您可以根据平板电脑的数量来设置FE的内存大小，如下表所示。例如，如果平板电脑的数量少于100万，您应为FE分配至少16GB的内存。您可以在**fe.conf**文件中的**JAVA_OPTS**配置项中配置参数`-Xms`和`-Xmx`的值，这两个参数的值应该一致。请注意，所有FE的配置应该相同，因为任何一个FE都可能被选为Leader。

|片剂数量|每个FE的内存大小|
|---|---|
|100万以下|16 GB|
|1～200万|32GB|
|2～500万|64GB|
|5～1000万|128GB|

## StarRocks如何计算其查询时间？

StarRocks支持使用多线程来查询数据。查询时间指的是多个线程查询数据所用的时间。

## StarRocks支持在本地导出数据时设置路径吗？

不支持。

## StarRocks的并发上限是多少？

您可以根据实际或模拟的业务场景来测试并发限制。根据一些用户的反馈，最高可达到20000 QPS或30000 QPS。

## 为什么StarRocks第一次SSB测试的性能比第二次慢？

第一次查询时读取磁盘的速度取决于磁盘的性能。第一次查询后，为后续查询生成了页面缓存，因此查询速度比之前快。

## 一个集群至少需要配置多少个BE？

StarRocks支持单节点部署，所以您至少需要配置一个BE。BE需要支持AVX2，因此我们建议您在至少拥有8核心和16GB或更高配置的机器上部署BE。

## 在使用Apache Superset可视化StarRocks中的数据时，如何设置数据权限？

您可以创建一个新的用户账户，然后通过授予用户对表查询的权限来设置数据权限。

## 为什么我将enable_profile设置为true后，profile无法显示？

报告只提交给领导者FE以供访问。

## 如何查看StarRocks表中的字段注释？

运行show create table xxx命令。

## 创建表时，如何为NOW()函数指定默认值？

只有StarRocks 2.1或更高版本支持为函数指定默认值。对于StarRocks 2.1之前的版本，您只能为函数指定常量值。

## 如何释放BE节点的存储空间？

您可以使用rm -rf命令删除trash目录。如果您已经从快照中恢复了数据，您可以删除snapshot目录。

## BE节点可以添加额外的磁盘吗？

可以。您可以将磁盘添加到BE配置项storage_root_path指定的目录中。

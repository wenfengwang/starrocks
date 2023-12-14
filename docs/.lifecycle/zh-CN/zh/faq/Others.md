---
displayed_sidebar: "Chinese"
---

# Other

This article summarizes other common questions when using StarRocks.

## Are VARCHAR(32) and STRING occupying the same storage space?

VARCHAR(32) and STRING are both variable-length data types. When storing data of the same length, VARCHAR(32) and STRING occupy the same storage space.

## Are there any performance differences between VARCHAR(32) and STRING when querying?

No, the performance is the same.

## How to handle the garbled TXT file exported from Oracle, even after modifying its character set to UTF-8?

Consider the file character set as `GBK` for character set conversion with the following steps:

1. For example, a file named **origin** is garbled. Use the following command to view its character set as `ISO-8859-1`.

    ```Plain_Text
    file --mime-encoding origin.txt
    origin.txt：iso-8859-1
    ```

2. Use the `iconv` command to convert the file's character set to UTF-8.

    ```Plain_Text
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. If the file still contains garbled characters after conversion, consider the file's character set as `GBK`, and then convert it to UTF-8.

    ```Shell
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## Are the string lengths defined in MySQL consistent with those defined in StarRocks?

In StarRocks, in VARCHAR(n), n represents the number of bytes, while in MySQL, in VARCHAR(n), n represents the number of characters. According to UTF-8, 1 Chinese character equals 3 bytes. When both StarRocks and MySQL define n as the same number, the number of characters saved in MySQL is 3 times that of StarRocks.

## Can the partition fields of a table be of type FLOAT, DOUBLE, or DECIMAL?

No, only DATE, DATETIME, and INT data types are supported.

## How to view the storage occupied by data in a table?

Execute the `SHOW DATA` statement to view the storage space occupied by data as well as the data volume, replica count, and row count.

> Note: Data import is not updated in real time. It may take about 1 minute after import to see the latest data.

## How to adjust the database quota of StarRocks?

Run the following code to adjust the database quota:

```SQL
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## Does StarRocks support partial field updates using the UPSERT syntax?

StarRocks 2.2 and above versions support partial field updates using the primary key model. StarRocks 1.9 and above versions support full field updates using the primary key model. For more information, see the [Primary Key Model](../table_design/table_types/primary_key_table.md) in the StarRocks 2.2 version.

## How to use the atomic replace table and atomic replace partition functions?

Execute the `SWAP WITH` statement to implement the atomic replace table and partition functions. The `SWAP WITH` statement is safer than the `INSERT OVERWRITE` statement. Before the atomic replacement, you can check the data to verify whether the replaced data is the same as the original data.

- Atomic replace table: For example, if there is a table named `table1` and you want to replace `table1` with another table, follow these steps:

    1. Create a new table named `table2`.

    ```SQL
    create table2 like table1;
    ```

    2. Use methods such as Stream Load, Broker Load, or Insert Into to import data from `table1` into the new table `table2`.
    3. Perform the atomic replacement of `table1` with `table2`.

    ```SQL
    ALTER TABLE table1 SWAP WITH table2;
    ```

    By doing this, the data will be accurately imported into `table1`.

- Atomic replace partition: For example, if there is a table named `table1` and you want to replace the partition data in `table1`, follow these steps:

    1. Create a temporary partition.

    ```SQL
    ALTER TABLE table1

    ADD TEMPORARY PARTITION tp1

    VALUES LESS THAN("2020-02-01");
    ```

    2. Import the partition data from `table1` into the temporary partition.
    3. Perform the atomic replace of the partition.

    ```SQL
    ALTER TABLE table1

    REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
    ```

## Error "error to open replicated environment, will exit" occurs when restarting FE

This error is caused by a vulnerability in BDBJE. Upgrading BDBJE to version 1.17 or higher can fix this problem.

## Error "Broker list path exception" occurs when querying a newly created Apache Hive™ table

### **Problem Description**

```Plain_Text
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### **Solution**

Contact StarRocks technical support to confirm whether the namenode's address and port are correct and whether you have permission to access the namenode's address and port.

## Error "get hive partition meta data failed" occurs when querying a newly created Apache Hive™ table

### **Problem Description**

```Plain_Text
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### **Solution**

Ensure that there is a network connection and transfer a copy of the **host** file from the cluster to each BE machine.

## Error "do_open failed. reason = Invalid ORC postscript length" occurs when accessing an ORC external table of Apache Hive™

### **Problem Description**

The metadata of Apache Hive™ is cached in the FE of StarRocks, but there is a two-hour time difference in the metadata update of StarRocks. If new data is inserted or updated in the Apache Hive™ table before StarRocks completes the update, the inconsistency between the data scanned by BE and the data obtained by FE in HDFS will cause this error.

```Plain_Text
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### **Solution**

There are two solutions:

- Upgrade StarRocks to version 2.2 or higher.
- Manually refresh the metadata of Apache Hive™ table. For more information, see [Cache Update](../data_source/External_table.md#手动更新元数据缓存).

## Error "caching_sha2_password cannot be loaded" occurs when connecting to a MySQL external table

### **Problem Description**

The default authentication method for MySQL 5.7 is `mysql_native_password`. Using the default authentication method of `caching_sha2_password` in MySQL 8.0 for authentication will cause a connection error.

### **Solution**

There are two solutions:

- Set the root user

```Plain_Text
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

- Modify the **my.cnf** file

```Plain_Text
vim my.cnf

[mysqld]

default_authentication_plugin=mysql_native_password
```

## Why is the disk space not immediately released after deleting a table?

执行 DROP TABLE 语句删表后需等待磁盘空间释放。如果想要快速释放磁盘空间可以使用 DROP TABLE FORCE 语句。执行 DROP TABLE FORCE 语句删除表时不会检查该表是否存在未完成的事务，而是直接将表删除。建议谨慎使用 DROP TABLE FORCE 语句，因为使用该语句删除的表不能恢复。

## 如何查看 StarRocks 的版本？

执行 `select current_version();` 命令或者CLI `sh bin/show_fe_version.sh` 命令查看版本。

## 如何设置 FE 的内存大小？

元数据信息都保存在 FE 的内存中。可以按下表所示参考 Tablet 的数量来设置 FE 的内存大小。例如 Tablet 数量为 100 万以下，则最少要分配 16 GB 的内存给 FE。您需要在 **fe.conf** 文件的 `JAVA_OPTS` 中通过配置 `-Xms` 和 `-Xmx` 参数来设置 FE 内存大小，并且两者取值保持一致即可。注意，集群中所有 FE 需要统一配置，因为每个 FE 都可能成为 Leader。

| Tablet 数量    | FE 内存大小 |
| -------------- | ----------- |
| 100 万以下     | 16 GB        |
| 100 万 ～ 200 万 | 32 GB        |
| 200 万 ～ 500 万 | 64 GB        |
| 500 万 ～ 1 千万   | 128 GB       |

## StarRocks 如何计算查询时间?

StarRocks 是多线程计算，查询时间即为查询最慢的线程所用的时间。

## StarRocks 支持导出数据到本地时设置路径吗？

不支持。

## StarRocks 的并发量级是多少？

建议根据业务场景或模拟业务场景测试 StarRocks 的并发量级。在部分客户的并发量级最高达到 20,000 QPS 或 30,000 QPS。

## 为什么 StarRocks 的 SSB 测试首次执行速度较慢，后续执行较快？

第一次查询读盘跟磁盘性能相关。第一次查询后系统的页面缓存生效，后续查询会先扫描页面缓存，所以速度有所提升。

## 一个集群最少可以配置多少个 BE？

StarRocks 支持单节点部署，所以 BE 最小配置个数是 1 个。BE 需要支持 AVX2 指令集，所以推荐部署 BE 的机器配置在 8 核 16 GB及以上。建议正常应用环境配置 3 个 BE。

## 使用 Apache Superset 框架呈现 StarRocks 中的数据时，如何进行数据权限配置？

创建一个新用户，然后通过给该用户授予表查询权限（SELECT）进行数据权限控制。

## 为什么将 `enable_profile` 指定为 `true` 后 profile 无法显示？

因为报告信息只汇报给主 FE，只有主 FE 可以查看报告信息。同时，如果通过 StarRocks Manager 查看 profile， 必须确保 FE 配置项 `enable_collect_query_detail_info` 为 `true`。

## 如何查看 StarRocks 表里的字段注释？

可以通过 `show create table xxx` 命令查看。

## 建表时可以指定 now() 函数的默认值吗？

StarRocks 2.1 及更高版本支持为函数指定默认值。低于 StarRocks 2.1 的版本仅支持为函数指定常量。

## StarRocks 外部表同步出错，应该如何解决？

**提示问题**：

SQL 错误 [1064] [42000]: data cannot be inserted into table with empty partition.Use `SHOW PARTITIONS FROM external_t` to see the currently partitions of this table.

查看Partitions时提示另一错误：SHOW PARTITIONS FROM external_t
SQL 错误 [1064] [42000]: Table[external_t] is not a OLAP/ELASTICSEARCH/HIVE table

**解决方法**：

原来是建外部表时端口不对，正确的端口是"port"="9020"，不是9931.

## 磁盘存储空间不足时，如何释放可用空间？

您可以通过 `rm -rf` 命令直接删除 `trash` 目录下的内容。在完成恢复数据备份后，您也可以通过删除 `snapshot` 目录下的内容释放存储空间。

## 磁盘存储空间不足，如何扩展磁盘空间？

如果 BE 节点存储空间不足，您可以在 BE 配置项 `storage_root_path` 所对应目录下直接添加磁盘。
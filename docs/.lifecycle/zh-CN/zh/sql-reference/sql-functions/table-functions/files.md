```yaml
      + {T}
      + {T}
    + {T}
  + {T}
```
```yaml
      + {T}
      + {T}
    + {T}
  + {T}
```
- `schema_auto_detect_sample_rows`：扫描每个采样数据文件中的数据行数。范围：[-1, 500]。如果将此参数设置为 `-1`，则扫描所有数据行。
- `schema_auto_detect_sample_files`：在每个批次中采样的随机数据文件数量。有效值：`1`（默认值）和 `-1`。如果将此参数设置为 `-1`，则扫描所有数据文件。

采样后，StarRocks 根据以下规则 Union 所有数据文件的列：

- 对于具有不同列名或索引的列，StarRocks 将每列识别为单独的列，最终返回所有单独列。
- 对于列名相同但数据类型不同的列，StarRocks 将这些列识别为相同的列，并为其选择一个通用的数据类型。例如，如果文件 A 中的列 `col1` 是 INT 类型，而文件 B 中的列 `col1` 是 DECIMAL 类型，则在返回的列中使用 DOUBLE 数据类型。STRING 类型可用于统一所有数据类型。

如果 StarRocks 无法统一所有列，将生成一个包含错误信息和所有文件 Schema 的错误报告。

> **注意**
>
> 单个批次中的所有数据文件必须为相同的文件格式。

-->

### unload_data_param

从 v3.2 版本开始，FILES() 支持在远程存储中定义可写入文件以进行数据导出。有关详细说明，请参阅[使用 INSERT INTO FILES 导出数据](../../../unloading/unload_using_insert_into_files.md)。

```sql
-- 自 v3.2 版本起支持。
unload_data_param::=
    "compression" = "<compression_method>",
    "max_file_size" = "<file_size>",
    "partition_by" = "<column_name> [, ...]" 
    "single" = { "true" | "false" } 
```

| **参数**          | **必填** | **说明**                                                          |
| ---------------- | ------------ | ------------------------------------------------------------ |
| compression      | 是          | 导出数据时要使用的压缩方法。有效值：<ul><li>`uncompressed`：不使用任何压缩算法。</li><li>`gzip`：使用 gzip 压缩算法。</li><li>`brotli`：使用 Brotli 压缩算法。</li><li>`zstd`：使用 Zstd 压缩算法。</li><li>`lz4`：使用 LZ4 压缩算法。</li></ul>                  |
| max_file_size    | 否           | 当数据导出为多个文件时，每个数据文件的最大大小。默认值：`1GB`。单位：B、KB、MB、GB、TB 和 PB。 |
| partition_by     | 否           | 用于将数据文件分区到不同存储路径的列，可以指定多个列。FILES() 提取指定列的 Key/Value 信息，并将数据文件存储在以对应 Key/Value 区分的子路径下。详细使用方法请见以下示例五。 |
| single           | 否           | 是否将数据导出到单个文件中。有效值：<ul><li>`true`：数据存储在单个数据文件中。</li><li>`false`（默认）：如果达到 `max_file_size`，则数据存储在多个文件中。</li></ul>                  |

> **注意**
>
> 不能同时指定 `max_file_size` 和 `single`。

## 注意事项

自 v3.2 版本起，除了基本数据类型，FILES() 还支持复杂数据类型 ARRAY、JSON、MAP 和 STRUCT。

## 示例

示例一：查询 AWS S3 存储桶 `inserttest` 内 Parquet 文件 **parquet/par-dup.parquet** 中的数据

```Plain
MySQL > SELECT * FROM FILES(
     "path" = "s3://inserttest/parquet/par-dup.parquet",
     "format" = "parquet",
     "aws.s3.access_key" = "XXXXXXXXXX",
     "aws.s3.secret_key" = "YYYYYYYYYY",
     "aws.s3.region" = "us-west-2"
);
+------+---------------------------------------------------------+
| c1   | c2                                                      |
+------+---------------------------------------------------------+
|    1 | {"1": "key", "1": "1", "111": "1111", "111": "aaaa"}    |
|    2 | {"2": "key", "2": "NULL", "222": "2222", "222": "bbbb"} |
+------+---------------------------------------------------------+
2 rows in set (22.335 sec)
```

示例二：将 AWS S3 存储桶 `inserttest` 内 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 中的数据插入至表 `insert_wiki_edit` 中：

```Plain
MySQL > INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 rows affected (23.03 sec)
{'label':'insert_d8d4b2ee-ac5c-11ed-a2cf-4e1110a8f63b', 'status':'VISIBLE', 'txnId':'2440'}
```

示例三：基于 AWS S3 存储桶 `inserttest` 内 Parquet 文件 **parquet/insert_wiki_edit_append.parquet** 中的数据创建表 `ctas_wiki_edit`：

```Plain
MySQL > CREATE TABLE ctas_wiki_edit AS
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
Query OK, 2 rows affected (22.09 sec)
{'label':'insert_1a217d70-2f52-11ee-9e4a-7a563fb695da', 'status':'VISIBLE', 'txnId':'3248'}
```

示例四：查询 HDFS 集群内 Parquet 文件 **/geo/country=US/city=LA/file1.parquet** 中的数据（其中仅包含两列 - `id` 和 `user`），并提取其路径中的 Key/Value 信息作为返回的列。

```Plain
SELECT * FROM FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/geo/country=US/city=LA/file1.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "columns_from_path" = "country, city"
);
+------+---------+---------+------+
| id   | user    | country | city |
+------+---------+---------+------+
|    1 | richard | US      | LA   |
|    2 | amber   | US      | LA   |
+------+---------+---------+------+
2 rows in set (3.84 sec)
```

示例五：将 `sales_records` 中的所有数据行导出为多个 Parquet 文件，存储在 HDFS 集群的路径 **/unload/partitioned/** 下。这些文件存储在不同的子路径中，这些子路径根据列 `sales_time` 中的值来区分。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/partitioned/",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "partition_by" = "sales_time"
)
SELECT * FROM sales_records;
```
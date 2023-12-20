
# 重试文件

## 描述

尝试重新加载管道中的所有数据文件或某个特定数据文件。

## 语法

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { RETRY ALL | RETRY FILE '<file_name>' }
```

## 参数

### pipe_name

管道的名称。

### file_name

您想要尝试重新加载的数据文件的存储路径。请注意，您必须提供该文件的完整存储路径。如果您指定的文件不属于您在 pipe_name 中指定的管道，系统将返回一个错误。

## 示例

以下示例展示了如何重试加载名为 user_behavior_replica 的管道中的所有数据文件：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY ALL;
```

以下示例展示了如何在名为 user_behavior_replica 的管道中重试加载数据文件 s3://starrocks-datasets/user_behavior_ten_million_rows.parquet：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY FILE 's3://starrocks-datasets/user_behavior_ten_million_rows.parquet';
```

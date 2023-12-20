---
displayed_sidebar: English
---

# 修改管道设置

## 描述

修改管道属性的配置。

## 语法

```SQL
ALTER PIPE [db_name.]<pipe_name> 
SET PROPERTY
(
    "<key>" = <value>[, "<key>" = "<value>" ...]
) 
```

## 参数

### db_name

管道所属的数据库名称。

### pipe_name

管道的名称。

### PROPERTIES

要更改的管道属性设置。格式：`"键" = "值"`。有关支持的属性的更多信息，请参见[CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)。

## 示例

在名为 mydatabase 的数据库中，将名为 user_behavior_replica 的管道的 AUTO_INGEST 属性设置改为 FALSE：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```

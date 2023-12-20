---
displayed_sidebar: English
---

# ALTER PIPE

## 描述

修改管道的属性设置。

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

您想要修改的管道属性。格式：`"key" = "value"`。有关支持的属性的更多信息，请参见 [CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)。

## 示例

将数据库 `mydatabase` 中名为 `user_behavior_replica` 的管道的 `AUTO_INGEST` 属性设置更改为 `FALSE`：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```
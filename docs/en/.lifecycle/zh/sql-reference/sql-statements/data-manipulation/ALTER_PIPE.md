---
displayed_sidebar: English
---

# 修改管道

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

管道所属的数据库的名称。

### pipe_name

管道的名称。

### 属性

要修改管道设置的属性。格式： `"key" = "value"`.有关支持的属性的详细信息，请参阅 [CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)。

## 例子

更改名为 `user_behavior_replica` 的管道在名为 `mydatabase` 的数据库中 `AUTO_INGEST` 属性的设置为 `FALSE`：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
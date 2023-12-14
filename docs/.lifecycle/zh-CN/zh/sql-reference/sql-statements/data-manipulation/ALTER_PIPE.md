---
displayed_sidebar: "中文"
---

# 更改管道

## 功能

修改管道的执行参数。

## 语法

```SQL
ALTER PIPE [db_name.]<pipe_name> 
SET
(
    "<key>" = <value>[, "<key>" = "<value>" ...]
) 
```

## 参数说明

### db_name

管道所属的数据库名称。

### pipe_name

管道名称。

### **属性**

要修改的执行参数设置。格式：`"key" = "value"`。有关支持的执行参数，请参见 [创建管道](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)。

## 示例

修改数据库 `mydatabase` 下名为 `user_behavior_replica` 的管道的 `AUTO_INGEST` 属性为 `FALSE`：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```
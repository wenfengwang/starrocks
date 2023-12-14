---
displayed_sidebar: "Chinese"
---

# ALTER PIPE(修改管道)

## Description(描述)

修改管道的属性设置。

## Syntax(语法)

```SQL
ALTER PIPE [db_name.]<pipe_name> 
SET PROPERTY
(
    "<key>" = <value>[, "<key>" = "<value>" ...]
) 
```

## Parameters(参数)

### db_name(数据库名称)

管道所属的数据库名称。

### pipe_name(管道名称)

管道的名称。

### PROPERTIES(属性)

要修改管道设置的属性。格式：`"key" = "value"`。有关支持的属性的更多信息，请参见[CREATE PIPE](../../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)。

## Examples(示例)

将名为`user_behavior_replica`的管道在名为`mydatabase`的数据库中的`AUTO_INGEST`属性设置更改为`FALSE`：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica
SET
(
    "AUTO_INGEST" = "FALSE"
);
```
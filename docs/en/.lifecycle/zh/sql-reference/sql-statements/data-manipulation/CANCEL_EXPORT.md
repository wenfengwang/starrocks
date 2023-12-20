---
displayed_sidebar: English
---

# 取消导出

## 描述

取消指定的数据卸载作业。状态为`CANCELLED`或`FINISHED`的卸载作业无法被取消。取消卸载作业是一个异步过程。您可以使用[SHOW EXPORT](../data-manipulation/SHOW_EXPORT.md)语句来检查卸载作业是否已成功取消。如果`State`的值为`CANCELLED`，则表示卸载作业已成功取消。

执行 CANCEL EXPORT 语句需要您至少拥有以下权限中的一个：`SELECT_PRIV`、`LOAD_PRIV`、`ALTER_PRIV`、`CREATE_PRIV`、`DROP_PRIV` 和 `USAGE_PRIV`，这些权限需针对卸载作业所属的数据库。有关权限描述的更多信息，请参阅[GRANT](../account-management/GRANT.md)。

## 语法

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## 参数

|**参数**|**是否必填**|**描述**|
|---|---|---|
|db_name|否|卸载作业所属的数据库名称。如果未指定此参数，则会取消当前数据库中的卸载作业。|
|query_id|是|卸载作业的查询ID。您可以使用[LAST_QUERY_ID()](../../sql-functions/utility-functions/last_query_id.md)函数来获取ID。请注意，该函数只返回最新的查询ID。|

## 示例

示例 1：取消当前数据库中查询ID为`921d8f80-7c9d-11eb-9342-acde48001121`的卸载作业。

```SQL
CANCEL EXPORT
WHERE QUERYID = "921d8f80-7c9d-11eb-9342-acde48001121";
```

示例 2：取消`example_db`数据库中查询ID为`921d8f80-7c9d-11eb-9342-acde48001122`的卸载作业。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE QUERYID = "921d8f80-7c9d-11eb-9342-acde48001122";
```
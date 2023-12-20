---
displayed_sidebar: English
---

# 取消导出

## 描述

此操作用于取消指定的数据卸载作业。已处于“`已取消（CANCELLED）`”或“`已完成（FINISHED）`”状态的卸载作业无法被取消。取消卸载作业是一个异步过程。您可以使用[SHOW EXPORT](../data-manipulation/SHOW_EXPORT.md)语句来检查卸载作业是否已经成功被取消。如果作业的状态（`State`）显示为“`已取消（CANCELLED）`”，则表示卸载作业已成功取消。

执行 CANCEL EXPORT 语句需要您至少拥有以下权限中的一种：`SELECT_PRIV`、`LOAD_PRIV`、`ALTER_PRIV`、`CREATE_PRIV`、`DROP_PRIV` 以及 `USAGE_PRIV`。要了解更多关于权限的描述，请参考 [GRANT](../account-management/GRANT.md) 文档。

## 语法

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## 参数

|参数|必填|说明|
|---|---|---|
|db_name|No|卸载作业所属的数据库的名称。如果未指定此参数，则取消当前数据库中的卸载作业。|
|query_id|Yes|卸载作业的查询ID。您可以使用 LAST_QUERY_ID() 函数获取 ID。请注意，此函数仅返回最新的查询 ID。|

## 示例

示例 1：取消当前数据库中查询 ID 为 921d8f80-7c9d-11eb-9342-acde48001121 的卸载作业。

```SQL
CANCEL EXPORT
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001121";
```

示例 2：取消 example_db 数据库中查询 ID 为 921d8f80-7c9d-11eb-9342-acde48001121 的卸载作业。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```

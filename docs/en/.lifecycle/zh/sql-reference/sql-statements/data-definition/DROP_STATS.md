---
displayed_sidebar: English
---

# 删除统计信息

## 描述

删除 CBO 统计信息，包括基本统计和直方图。有关详细信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)。

您可以删除不需要的统计信息。删除统计信息时，将同时删除数据、元数据以及过期缓存中的统计信息。请注意，如果自动收集任务正在进行中，先前删除的统计信息可能会被再次收集。您可以使用 `SHOW ANALYZE STATUS` 查看收集任务的历史记录。

此语句从 v2.4 版本开始支持。

## 语法

### 删除基本统计

```SQL
DROP STATS tbl_name
```

### 删除直方图

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 引用

有关收集 CBO 统计信息的更多信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。
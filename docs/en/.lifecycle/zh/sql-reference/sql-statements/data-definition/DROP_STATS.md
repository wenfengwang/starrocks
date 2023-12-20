---
displayed_sidebar: English
---

# 删除统计数据

## 描述

删除CBO统计信息，包括基础统计和直方图。更多信息，请参见[为CBO收集统计信息](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)。

您可以删除不再需要的统计信息。当您删除统计信息时，会同时删除统计数据和元数据，以及过期缓存中的统计信息。请注意，如果自动收集任务正在进行，可能会重新收集之前删除的统计信息。您可以使用 `SHOW ANALYZE STATUS` 来查看收集任务的历史。

该语句从v2.4版本开始支持。

## 语法

### 删除基础统计

```SQL
DROP STATS tbl_name
```

### 删除直方图

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 参考资料

更多关于为CBO收集统计信息的信息，请参见[为CBO收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。
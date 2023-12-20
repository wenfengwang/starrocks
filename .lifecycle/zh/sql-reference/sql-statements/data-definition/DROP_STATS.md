---
displayed_sidebar: English
---

# 删除统计数据

## 说明

删除CBO统计信息，包括基本统计数据和直方图。更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)。

您可以移除不再需要的统计信息。当您删除统计数据时，相关的数据和元数据将一并被清除，包括已过期缓存中的统计信息。请注意，如果自动收集任务正在执行，可能会重新收集之前已删除的统计信息。您可以通过 SHOW ANALYZE STATUS 命令查看收集任务的历史。

此语句从v2.4版本开始支持。

## 语法

### 删除基础统计数据

```SQL
DROP STATS tbl_name
```

### 删除直方图

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 参考资料

要了解更多关于为CBO收集统计信息的内容，请参阅[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。

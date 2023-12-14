---
displayed_sidebar: "中文"
---

# 删除统计信息

## 描述

删除CBO统计信息，包括基本统计信息和直方图。有关更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)。

您可以删除您不需要的统计信息。当您删除统计信息时，数据和统计信息的元数据以及缓存中的统计信息都将被删除。请注意，如果自动收集任务正在进行中，先前删除的统计信息可能会被再次收集。您可以使用 `SHOW ANALYZE STATUS` 命令查看收集任务的历史记录。

此语句从v2.4版本开始支持。

## 语法

### 删除基本统计信息

```SQL
DROP STATS tbl_name
```

### 删除直方图

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 参考

有关收集CBO统计信息的更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。
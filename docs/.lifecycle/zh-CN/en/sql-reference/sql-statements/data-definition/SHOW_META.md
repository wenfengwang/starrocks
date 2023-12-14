---
displayed_sidebar: "English"
---

# 显示元数据

## 描述

查看CBO统计信息的元数据，包括基本统计信息和直方图。

此语句支持从v2.4版本开始。

### 查看基本统计信息的元数据

#### 语法

```SQL
SHOW STATS META [WHERE]
```

此语句返回以下列。

| **列名**    | **描述**                           |
| ----------- | -------------------------------- |
| Database    | 数据库名称。                       |
| Table       | 表名称。                           |
| Columns     | 列名称。                          |
| Type        | 统计类型。`FULL`表示完整收集，`SAMPLE`表示抽样收集。 |
| UpdateTime  | 当前表的最新统计信息更新时间。    |
| Properties  | 自定义参数。                       |
| Healthy     | 统计信息的健康状况。               |

### 查看直方图的元数据

#### 语法

```SQL
SHOW HISTOGRAM META [WHERE];
```

此语句返回以下列。

| **列名**    | **描述**                           |
| ----------- | -------------------------------- |
| Database    | 数据库名称。                       |
| Table       | 表名称。                           |
| Column      | 列。                              |
| Type        | 统计类型。直方图的值为`HISTOGRAM`。|
| UpdateTime  | 当前表的最新统计信息更新时间。    |
| Properties  | 自定义参数。                       |

## 参考

有关为CBO收集统计信息的更多信息，请参阅[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。
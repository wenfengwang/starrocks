---
displayed_sidebar: English
---

# 查看元数据

## 描述

查看CBO统计信息的元数据，包括基础统计数据和直方图。

此语句从v2.4版本开始支持。

### 查看基础统计数据的元数据

#### 语法

```SQL
SHOW STATS META [WHERE]
```

此语句返回以下列信息。

|专栏|说明|
|---|---|
|数据库|数据库名称。|
|表|表名称。|
|列|列名称。|
|类型|统计数据的类型。 FULL 表示完整集合，SAMPLE 表示采样集合。|
|UpdateTime|当前表的最新统计更新时间。|
|属性|自定义参数。|
|健康|健康状况统计信息。|

### 查看直方图的元数据

#### 语法

```SQL
SHOW HISTOGRAM META [WHERE];
```

此语句返回以下列信息。

|专栏|描述|
|---|---|
|数据库|数据库名称。|
|表|表名称。|
|列|列。|
|类型|统计类型。该值为直方图的 HISTOGRAM。|
|UpdateTime|当前表的最新统计更新时间。|
|属性|自定义参数。|

## 参考资料

有关为CBO收集统计信息的更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。

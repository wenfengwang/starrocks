---
displayed_sidebar: English
---

# 取消加载

## 描述

取消给定的加载作业：[Broker Load](../data-manipulation/BROKER_LOAD.md)、[Spark Load](../data-manipulation/SPARK_LOAD.md) 或 [INSERT](./INSERT.md)。处于 `PREPARED`、`CANCELLED` 或 `FINISHED` 状态的加载作业无法被取消。

取消加载作业是一个异步过程。您可以使用[SHOW LOAD](../data-manipulation/SHOW_LOAD.md)语句来检查加载作业是否成功取消。如果 `State` 的值为 `CANCELLED`，并且 `type` 的值（显示在 `ErrorMsg`中）为 `USER_CANCEL`，则加载作业将成功取消。

## 语法

```SQL
CANCEL LOAD
[FROM db_name]
WHERE LABEL = "label_name"
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | 否           | 加载作业所属的数据库名称。如果未指定此参数，则默认取消当前数据库中的加载作业。 |
| label_name    | 是          | 加载作业的标签。                                   |

## 例子

示例1：取消当前数据库中标签为 `example_label` 的加载作业。

```SQL
CANCEL LOAD
WHERE LABEL = "example_label";
```

示例2：取消数据库 `example_db` 中标签为 `example_label` 的加载作业。

```SQL
CANCEL LOAD
FROM example_db
WHERE LABEL = "example_label";
```

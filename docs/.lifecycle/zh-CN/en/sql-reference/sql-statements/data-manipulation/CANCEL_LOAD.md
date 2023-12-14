---
displayed_sidebar: "Chinese"
---

# 取消加载

## 描述

取消给定的加载作业：[经纪人加载](../data-manipulation/BROKER_LOAD.md)，[Spark加载](../data-manipulation/SPARK_LOAD.md) 或 [INSERT](./INSERT.md)。处于 `PREPARED`，`CANCELLED` 或 `FINISHED` 状态的加载作业无法取消。

取消加载作业是一个异步过程。您可以使用 [SHOW LOAD](../data-manipulation/SHOW_LOAD.md) 语句来检查加载作业是否成功取消。如果 `State` 的值为 `CANCELLED`，`type` 的值（显示在 `ErrorMsg` 中）为 `USER_CANCEL`，则加载作业被成功取消。

## 语法

```SQL
CANCEL LOAD
[FROM 数据库名称]
WHERE LABEL = "标签名称"
```

## 参数

| **参数**     | **必需**     | **描述**                                                     |
| ------------- | ------------ | ------------------------------------------------------------ |
| 数据库名称   | 否           | 加载作业所属的数据库的名称。如果没有指定此参数，默认情况下将取消当前数据库中的加载作业。 |
| 标签名称     | 是           | 加载作业的标签。                                             |

## 示例

示例 1：取消当前数据库中标签为 `example_label` 的加载作业。

```SQL
CANCEL LOAD
WHERE LABEL = "example_label";
```

示例 2：取消 `example_db` 数据库中标签为 `example_label` 的加载作业。

```SQL
CANCEL LOAD
FROM example_db
WHERE LABEL = "example_label";
```
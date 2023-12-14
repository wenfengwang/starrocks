---
displayed_sidebar: "Chinese"
---

# 取消加载

## 功能

取消指定的[Broker Load](../data-manipulation/BROKER_LOAD.md)、[Spark Load](../data-manipulation/SPARK_LOAD.md)或[INSERT](./INSERT.md)导入作业。状态为PREPARED、CANCELLED或FINISHED的导入作业不能取消。

CANCEL LOAD是一个异步操作，执行后可以使用[SHOW LOAD](../data-manipulation/SHOW_LOAD.md)语句查看是否成功取消。当状态(`State`)为`CANCELLED`且导入作业失败原因(`ErrorMsg`)中的失败类型(`type`)为`USER_CANCEL`时，表示成功取消了导入作业。

## 语法

```SQL
CANCEL LOAD
[FROM db_name]
WHERE LABEL = "label_name"
```

## 参数说明

| **参数**   | **必选** | **说明**                                       |
| ---------- | -------- | ---------------------------------------------- |
| db_name    | 否       | 导入作业所在的数据库的名称。默认为当前数据库。 |
| label_name | 是       | 导入作业的标签。                                |

## 示例

示例一：取消当前数据库中标签为`example_label`的导入作业。

```SQL
CANCEL LOAD
WHERE LABEL = "example_label";
```

示例二：取消数据库`example_db`中标签为`example_label`的导入作业。

```SQL
CANCEL LOAD
FROM example_db
WHERE LABEL = "example_label";
```
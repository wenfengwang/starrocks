---
displayed_sidebar: English
---

# 取消加载

## 描述

取消指定的加载作业：[Broker Load](../data-manipulation/BROKER_LOAD.md)、[Spark Load](../data-manipulation/SPARK_LOAD.md) 或 [INSERT](./INSERT.md)。处于`PREPARED`、`CANCELLED`或`FINISHED`状态的加载作业无法被取消。

取消加载作业是一个异步过程。您可以使用[SHOW LOAD](../data-manipulation/SHOW_LOAD.md)语句来检查加载作业是否已经成功取消。如果`State`的值为`CANCELLED`且`ErrorMsg`中显示的`type`的值为`USER_CANCEL`，则表示加载作业已成功取消。

## 语法

```SQL
CANCEL LOAD
[FROM db_name]
WHERE LABEL = "label_name"
```

## 参数

|**参数**|**是否必填**|**描述**|
|---|---|---|
|db_name|否|加载作业所属的数据库名称。如果未指定此参数，默认会取消当前数据库中的加载作业。|
|label_name|是|加载作业的标签。|

## 示例

示例 1：取消当前数据库中标签为`example_label`的加载作业。

```SQL
CANCEL LOAD
WHERE LABEL = "example_label";
```

示例 2：取消`example_db`数据库中标签为`example_label`的加载作业。

```SQL
CANCEL LOAD
FROM example_db
WHERE LABEL = "example_label";
```
---
displayed_sidebar: English
---

# ALTER LOAD

## 描述

更改处于 **QUEUEING** 或 **LOADING** 状态的 Broker Load 作业的优先级。该语句自 v2.5 版本起支持。

> **注意**
> 更改处于 **LOADING** 状态的 Broker Load 作业的优先级不会影响作业的执行。

## 语法

```SQL
ALTER LOAD FOR <label_name>
PROPERTIES
(
    'priority'='{LOWEST | LOW | NORMAL | HIGH | HIGHEST}'
)
```

## 参数

|**参数**|**是否必填**|**描述**|
|---|---|---|
|label_name|是|负载作业的标签。格式：`[<database_name>.]<label_name>`。参见 [BROKER LOAD](../data-manipulation/BROKER_LOAD.md#database_name-and-label_name)。|
|priority|是|您想为负载作业指定的新优先级。有效值：`LOWEST`、`LOW`、`NORMAL`、`HIGH` 和 `HIGHEST`。参见 [BROKER LOAD](../data-manipulation/BROKER_LOAD.md)。|

## 示例

假设您有一个标签为 `test_db.label1` 的 Broker Load 作业，该作业处于 **QUEUEING** 状态。如果您希望该作业尽快执行，您可以运行以下命令将作业的优先级更改为 `HIGHEST`：

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```
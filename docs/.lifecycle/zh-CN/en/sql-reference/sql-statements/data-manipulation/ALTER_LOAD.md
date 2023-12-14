```yaml
---
displayed_sidebar: "Chinese"
---

# 修改LOAD

## 描述

更改处于**排队**或**加载**状态的Broker Load作业的优先级。此语句自v2.5以后受支持。

> **注意**
>
> 更改处于**加载**状态的Broker Load作业的优先级不会影响作业的执行。

## 语法

```SQL
ALTER LOAD FOR <label_name>
PROPERTIES
(
    'priority'='{LOWEST | LOW | NORMAL | HIGH | HIGHEST}'
)
```

## 参数

| **参数**      | **是否必需** | 描述                                                         |
| ------------- | ------------ | ------------------------------------------------------------ |
| label_name    | 是           | 加载作业的标签。格式：`[<database_name>.]<label_name>`。请参阅[BROKER LOAD](../data-manipulation/BROKER_LOAD.md#database_name-and-label_name)。 |
| priority      | 是           | 您要为加载作业指定的新优先级。有效值：`LOWEST`、`LOW`、`NORMAL`、`HIGH` 和 `HIGHEST`。请参阅[BROKER LOAD](../data-manipulation/BROKER_LOAD.md)。 |

## 示例

假设您有一个标签为`test_db.label1`且该作业处于**排队**状态的Broker Load作业。如果您希望尽快运行该作业，可以运行以下命令将作业的优先级更改为`HIGHEST`：

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```
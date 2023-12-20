---
displayed_sidebar: English
---

# 调整负载作业优先级

## 描述

此命令用于更改处于排队（**QUEUEING**）或加载（**LOADING**）状态的Broker Load作业的优先级。该语句自v2.5版本起支持。

> **注意**
> 更改正在加载（**LOADING**）状态中的Broker Load作业的优先级并不会影响作业的执行进程。

## 语法

```SQL
ALTER LOAD FOR <label_name>
PROPERTIES
(
    'priority'='{LOWEST | LOW | NORMAL | HIGH | HIGHEST}'
)
```

## 参数

|参数|必填|说明|
|---|---|---|
|label_name|是|加载作业的标签。格式：[<数据库名称>.]<标签名称>。请参阅经纪人负载。|
|priority|Yes|您要为加载作业指定的新优先级。有效值：最低、低、正常、高和最高。请参阅经纪人负载。|

## 示例

假设您有一个标签为`test_db.label1`的Broker Load作业，该作业当前处于**QUEUEING**状态。如果您希望该作业能尽快执行，您可以运行以下命令，将作业的优先级改为`HIGHEST`：

```SQL
ALTER LOAD FOR test_db.label1
PROPERTIES
(
    'priority'='HIGHEST'
);
```

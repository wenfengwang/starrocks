---
displayed_sidebar: "Chinese"
---

# 显示事务

## 功能

此语法用于查看指定事务ID的事务详情。

## 语法

```sql
SHOW TRANSACTION
[FROM <db_name>]
WHERE id = <transaction_id>
```

返回结果示例：

```plain text
TransactionId: 4005
Label: insert_8d807d5d-bcdd-46eb-be6d-3fa87aa4952d
Coordinator: FE: 10.74.167.16
TransactionStatus: VISIBLE
LoadJobSourceType: INSERT_STREAMING
PrepareTime: 2020-01-09 14:59:07
CommitTime: 2020-01-09 14:59:09
FinishTime: 2020-01-09 14:59:09
Reason:
ErrorReplicasCount: 0
ListenerId: -1
TimeoutMs: 300000
```

* TransactionId：事务 id
* Label：导入任务对应的标签
* Coordinator：事务协调器
* TransactionStatus：事务状态
* PREPARE：准备阶段
* COMMITTED：事务成功，但数据不可见
* VISIBLE：事务成功且数据可见
* ABORTED：事务失败
* LoadJobSourceType：导入任务类型
* PrepareTime：事务开始时间
* CommitTime：事务提交成功时间
* FinishTime：数据可见时间
* Reason：错误信息
* ErrorReplicasCount：错误的副本数量
* ListenerId：相关导入作业的 id
* TimeoutMs：事务超时时间，单位毫秒

## 示例

1. 查看事务ID为4005的事务：

    ```sql
    SHOW TRANSACTION WHERE ID=4005;
    ```

2. 指定数据库中，查看事务ID为4005的事务：

    ```sql
    SHOW TRANSACTION FROM <db_name> WHERE ID=4005;
    ```
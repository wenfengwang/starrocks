---
displayed_sidebar: English
---

# 显示交易明细

## 描述

此语法用于查看指定交易 ID 的交易详情。

语法：

```sql
SHOW TRANSACTION
[FROM <db_name>]
WHERE id = transaction_id
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

* TransactionId：交易 ID
* Label：导入任务对应的标签
* Coordinator：负责交易协调的节点
* TransactionStatus：事务状态
* PREPARE：准备阶段
* COMMITTED：事务成功，但数据不可见
* VISIBLE：事务成功，数据可见
* ABORTED：事务失败
* LoadJobSourceType：导入任务的类型。
* PrepareTime：事务开始时间
* CommitTime：事务成功提交时间
* FinishTime：数据可见时间
* Reason：错误原因
* ErrorReplicasCount：有错误的副本数
* ListenerId：相关导入作业的 ID
* TimeoutMs：事务超时，毫秒为单位

## 例子

1. 查看 ID 为 4005 的交易：

    ```sql
    SHOW TRANSACTION WHERE ID=4005;
    ```

2. 在指定的数据库中，查看 ID 为 4005 的交易：

    ```sql
    SHOW TRANSACTION FROM db WHERE ID=4005;
    ```

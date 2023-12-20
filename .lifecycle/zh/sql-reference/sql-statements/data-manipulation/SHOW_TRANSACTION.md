---
displayed_sidebar: English
---

# 查看交易详情

## 描述

此语法用于查看指定交易ID的详细信息。

语法：

```sql
SHOW TRANSACTION
[FROM <db_name>]
WHERE id = transaction_id
```

返回结果示例：

```plain
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

* TransactionId：交易ID
* Label：与任务对应的标签
* Coordinator：负责事务协调的节点
* TransactionStatus：交易状态
* PREPARE：准备阶段
* COMMITTED：事务已成功，但数据尚未可见
* VISIBLE：事务已成功，数据已可见
* ABORTED：事务失败
* LoadJobSourceType：导入任务的类型
* PrepareTime：交易开始时间
* CommitTime：事务成功提交的时间
* FinishTime：数据变得可见的时间
* Reason：错误信息
* ErrorReplicasCount：出错的副本数
* ListenerId：相关导入作业的ID
* TimeoutMs：事务超时时间，以毫秒为单位

## 示例

1. 要查看ID为4005的交易：

   ```sql
   SHOW TRANSACTION WHERE ID=4005;
   ```

2. 在指定的数据库中查看ID为4005的交易：

   ```sql
   SHOW TRANSACTION FROM db WHERE ID=4005;
   ```

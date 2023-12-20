---
displayed_sidebar: English
---

# 显示更改物化视图的状态

## 描述

展示同步物化视图的构建状态。

:::提示

此操作不需要特定权限。

:::

## 语法

```SQL
SHOW ALTER MATERIALIZED VIEW [ { FROM | IN } db_name]
```

方括号[]内的参数是可选的。

## 参数

|参数|必填|说明|
|---|---|---|
|db_name|no|物化视图所在的数据库的名称。如果不指定该参数，则默认使用当前数据库。|

## 返回值

|返回|说明|
|---|---|
|JobId|刷新作业的 ID。|
|TableName|表的名称。|
|CreateTime|创建刷新作业的时间。|
|FinishedTime|刷新作业完成的时间。|
|BaseIndexName|基表的名称。|
|RollupIndexName|物化视图的名称。|
|RollupId|物化视图汇总的 ID。|
|TransactionId|等待执行的事务ID。|
|状态|作业的状态。|
|消息|错误消息。|
|进度|刷新作业的进度。|
|超时|刷新作业超时。|

## 示例

示例1：查看同步物化视图的构建状态。

```Plain
MySQL > SHOW ALTER MATERIALIZED VIEW\G
*************************** 1. row ***************************
          JobId: 475991
      TableName: lineorder
     CreateTime: 2022-08-24 19:46:53
   FinishedTime: 2022-08-24 19:47:15
  BaseIndexName: lineorder
RollupIndexName: lo_mv_sync_1
       RollupId: 475992
  TransactionId: 33067
          State: FINISHED
            Msg: 
       Progress: NULL
        Timeout: 86400
*************************** 2. row ***************************
          JobId: 477337
      TableName: lineorder
     CreateTime: 2022-08-24 19:47:25
   FinishedTime: 2022-08-24 19:47:45
  BaseIndexName: lineorder
RollupIndexName: lo_mv_sync_2
       RollupId: 477338
  TransactionId: 33068
          State: FINISHED
            Msg: 
       Progress: NULL
        Timeout: 86400
2 rows in set (0.00 sec)
```

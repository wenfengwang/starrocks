---
displayed_sidebar: English
---

# 终止

## 描述

终止在 StarRocks 中执行的线程当前正在进行的连接或查询。

:::提示

该操作不需要特殊权限。

:::

## 语法

```SQL
KILL [ CONNECTION | QUERY ] <processlist_id>
```

## 参数

|参数|说明|
|---|---|
|修饰符:CONNECTIONQUERY|使用 CONNECTION 修饰符，KILL 语句在终止连接正在执行的任何语句后终止与给定 processlist_id 关联的连接。使用 QUERY 修饰符，KILL 语句终止连接当前正在执行的语句，但保留连接本身完好无损。如果没有修饰符，则默认为 CONNECTION。|
|processlist_id|要终止的线程的 ID。您可以使用 SHOW PROCESSLIST 获取正在执行的线程的 ID。|

## 示例

```Plain
mysql> SHOW FULL PROCESSLIST;
+------+------+---------------------+--------+---------+---------------------+------+-------+-----------------------+-----------+
| Id   | User | Host                | Db     | Command | ConnectionStartTime | Time | State | Info                  | IsPending |
+------+------+---------------------+--------+---------+---------------------+------+-------+-----------------------+-----------+
|   20 | root | xxx.xx.xxx.xx:xxxxx | sr_hub | Query   | 2023-01-05 16:30:19 |    0 | OK    | show full processlist | false     |
+------+------+---------------------+--------+---------+---------------------+------+-------+-----------------------+-----------+
1 row in set (0.01 sec)

mysql> KILL 20;
Query OK, 0 rows affected (0.00 sec)
```

---
displayed_sidebar: English
---

# KILL

## 描述

终止 StarRocks 中正在执行的线程当前正在进行的连接或查询。

:::tip

此操作不需要权限。

:::

## 语法

```SQL
KILL [ CONNECTION | QUERY ] <processlist_id>
```

## 参数

|**参数**|**说明**|
|---|---|
|修饰符:<ul><li>CONNECTION</li><li>QUERY</li></ul>|<ul><li>使用 `CONNECTION` 修饰符，KILL 语句会在终止连接正在执行的任何语句后，终止与给定 `processlist_id` 关联的连接。</li><li>使用 `QUERY` 修饰符，KILL 语句仅终止连接当前正在执行的语句，但保留连接本身。</li><li>如果没有指定修饰符，缺省值为 `CONNECTION`。</li></ul>|
|processlist_id|您想要终止的线程的 ID。您可以使用 [SHOW PROCESSLIST](../Administration/SHOW_PROCESSLIST.md) 获取正在执行的线程的 ID。|

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
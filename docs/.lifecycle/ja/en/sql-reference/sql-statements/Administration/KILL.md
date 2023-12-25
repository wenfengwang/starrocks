---
displayed_sidebar: English
---

# KILL

## 説明

StarRocks内で実行中のスレッドが現在実行している接続またはクエリを終了します。

:::tip

この操作には権限は必要ありません。

:::

## 構文

```SQL
KILL [ CONNECTION | QUERY ] <processlist_id>
```

## パラメーター

| **パラメーター**            | **説明**                                              |
| ------------------------ | ------------------------------------------------------------ |
| 修飾子:<ul><li>CONNECTION</li><li>QUERY</li></ul> | <ul><li>`CONNECTION` 修飾子を使用すると、KILL ステートメントは、`processlist_id`に関連付けられた接続を、接続が実行中のステートメントを終了した後に終了します。</li><li>`QUERY` 修飾子を使用すると、KILL ステートメントは接続が現在実行中のステートメントを終了しますが、接続自体は維持されます。</li><li>修飾子が指定されていない場合、デフォルトは`CONNECTION`です。</li></ul> |
| processlist_id           | 終了したいスレッドのIDです。実行中のスレッドのIDは、[SHOW PROCESSLIST](../Administration/SHOW_PROCESSLIST.md)を使用して取得できます。 |

## 例

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

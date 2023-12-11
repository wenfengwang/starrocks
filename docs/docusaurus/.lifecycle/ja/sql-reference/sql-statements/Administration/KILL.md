---
displayed_sidebar: "Japanese"
---

# KILL（キル）

## 説明

StarRocks内で実行されているスレッドによって実行されている接続またはクエリを終了します。

## 構文

```SQL
KILL [ CONNECTION | QUERY ] <processlist_id>
```

## パラメータ

| **パラメータ**     | **説明**                                                  |
| ------------------------ | ------------------------------------------------------------ |
| Modifier:<ul><li>CONNECTION</li><li>QUERY</li></ul> | <ul><li>「CONNECTION」修飾子を使用すると、KILLステートメントは指定された「processlist_id」と関連付けられた接続を終了し、接続が実行している任意のステートメントを終了します。</li><li>「QUERY」修飾子を使用すると、KILLステートメントは接続が現在実行しているステートメントを終了しますが、接続自体はそのままにします。</li><li>修飾子が指定されていない場合、デフォルトは「CONNECTION」です。</li></ul> |
| processlist_id      | 終了したいスレッドのID。実行中のスレッドのIDは、[SHOW PROCESSLIST](../Administration/SHOW_PROCESSLIST.md)を使用して取得できます。 |

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
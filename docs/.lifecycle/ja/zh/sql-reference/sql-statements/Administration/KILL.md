---
displayed_sidebar: Chinese
---

# KILL

## 機能

現在のクラスタ内で実行中の特定の接続またはクエリを終了します。

:::tip

この操作には権限は必要ありません。

:::

## 文法

```SQL
KILL [ CONNECTION | QUERY ] <processlist_id>
```

## パラメータ説明

| **パラメータ**                | **説明**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| 修飾子：<ul><li>CONNECTION</li><li>QUERY</li></ul> | <ul><li>`CONNECTION` 修飾子を使用すると、KILL 文はまず `processlist_id` に関連する接続で実行中のステートメントを終了し、その後接続を終了します。</li><li>`QUERY` 修飾子を使用すると、KILL 文は `processlist_id` に関連する接続で実行中のステートメントのみを終了し、接続は終了しません。</li><li>修飾子を指定しない場合、デフォルトは `CONNECTION` です。</li></ul> |
| processlist_id          | 終了させるスレッドの ID。[SHOW PROCESSLIST](../Administration/SHOW_PROCESSLIST.md) を使用して、実行中のスレッドの ID を取得できます。 |

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

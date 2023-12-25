---
displayed_sidebar: Chinese
---

# SHOW VARIABLES

## 機能

現在のクラスターのシステム変数情報を表示します。

:::tip

この操作には権限が必要ありません。

:::

## 文法

```SQL
SHOW [ GLOBAL | SESSION ] VARIABLES
    [ LIKE <pattern> | WHERE <expr> ]
```

## パラメータ説明

| **パラメータ**              | **説明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| 修飾子：<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>`GLOBAL` 修飾子を使用すると、この文はグローバルシステム変数の値を表示します。新しい接続が生成されると、StarRocksはすべての変数のセッション値をグローバル値として自動的に初期化します。変数にグローバル値がない場合は、値は表示されません。</li><li>`SESSION` 修飾子を使用すると、この文は現在の接続に有効なシステム変数の値を表示します。変数にセッション値がない場合は、グローバル値が表示されます。`LOCAL` を `SESSION` の代わりに使用することもできます。</li><li>修飾子を指定しない場合、デフォルト値は `SESSION` です。</li></ul> |
| pattern               | 変数名にマッチする LIKE 句のパターンです。パターンには `%` ワイルドカードを使用できます。 |
| expr                  | 変数名 `variable_name` または変数値 `value` にマッチする WHERE 句の式です。 |

## 戻り値

| **戻り値**      | **説明** |
| ------------- | -------- |
| Variable_name | 変数名。 |
| Value         | 変数値。 |

## 例

例1：LIKE 句を使用して変数名に正確にマッチして変数情報を表示します。

```Plain
mysql> SHOW VARIABLES LIKE 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.01 sec)
```

例2：LIKE 句と `%` ワイルドカードを使用して変数名に曖昧にマッチして変数情報を表示します。

```Plain
mysql> SHOW VARIABLES LIKE '%imeou%';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| interactive_timeout                | 3600  |
| net_read_timeout                   | 60    |
| net_write_timeout                  | 60    |
| new_planner_optimize_timeout       | 3000  |
| query_delivery_timeout             | 300   |
| query_queue_pending_timeout_second | 300   |
| query_timeout                      | 300   |
| wait_timeout                       | 28800 |
+------------------------------------+-------+
9 rows in set (0.00 sec)
```

例3：WHERE 句を使用して変数名に正確にマッチして変数情報を表示します。

```Plain
mysql> show variables where variable_name = 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.17 sec)
```

例4：WHERE 句を使用して変数値に正確にマッチして変数情報を表示します。

```Plain
mysql> show variables where value = '28800';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.70 sec)
```

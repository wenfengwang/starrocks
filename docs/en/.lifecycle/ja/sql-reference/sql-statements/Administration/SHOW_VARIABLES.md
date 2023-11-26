---
displayed_sidebar: "Japanese"
---

# SHOW VARIABLES

## 説明

StarRocksのシステム変数を表示します。システム変数の詳細については、[システム変数](../../../reference/System_variable.md)を参照してください。

## 構文

```SQL
SHOW [ GLOBAL | SESSION ] VARIABLES
    [ LIKE <pattern> | WHERE <expr> ]
```

## パラメータ

| **パラメータ**          | **説明**                                              |
| ---------------------- | ------------------------------------------------------------ |
| Modifier:<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>`GLOBAL`修飾子を使用すると、ステートメントはグローバルシステム変数の値を表示します。これらは、StarRocksへの新しい接続の対応するセッション変数を初期化するために使用される値です。変数にグローバル値がない場合、値は表示されません。</li><li>`SESSION`修飾子を使用すると、ステートメントは現在の接続に対して有効なシステム変数の値を表示します。変数にセッション値がない場合、グローバル値が表示されます。`LOCAL`は`SESSION`の同義語です。</li><li>修飾子が指定されていない場合、デフォルトは`SESSION`です。</li></ul> |
| pattern                | LIKE句で変数名によって変数を一致させるために使用するパターンです。このパラメータにはワイルドカード%を使用できます。 |
| expr                   | WHERE句で変数名`variable_name`または変数値`value`によって変数を一致させるために使用する式です。 |

## 戻り値

| **戻り値**    | **説明**            |
| ------------- | -------------------------- |
| Variable_name | 変数の名前です。  |
| Value         | 変数の値です。 |

## 例

例1: LIKE句を使用して変数名を完全に一致させて変数を表示します。

```Plain
mysql> SHOW VARIABLES LIKE 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.01 sec)
```

例2: LIKE句とワイルドカード(%)を使用して変数名をおおよそ一致させて変数を表示します。

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
| tx_visible_wait_timeout            | 10    |
| wait_timeout                       | 28800 |
+------------------------------------+-------+
9 rows in set (0.00 sec)
```

例3: WHERE句を使用して変数名を完全に一致させて変数を表示します。

```Plain
mysql> show variables where variable_name = 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.17 sec)
```

例4: WHERE句を使用して変数の値を完全に一致させて変数を表示します。

```Plain
mysql> show variables where value = '28800';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.70 sec)
```

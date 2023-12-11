---
displayed_sidebar: "Japanese"
---

# 変数を表示

## 説明

StarRocksのシステム変数を表示します。システム変数の詳細については、[システム変数](../../../reference/System_variable.md)を参照してください。

## 構文

```SQL
SHOW [ GLOBAL | SESSION ] VARIABLES
    [ LIKE <pattern> | WHERE <expr> ]
```

## パラメータ

| **パラメータ**         | **説明**                                                |
| ---------------------- | ------------------------------------------------------ |
| Modifier:<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>`GLOBAL`修飾子を使用すると、ステートメントはグローバルシステム変数の値を表示します。これらは、StarRocksへの新しい接続の対応するセッション変数を初期化するために使用される値です。変数にグローバル値がない場合は、値は表示されません。</li><li>`SESSION`修飾子を使用すると、現在の接続に対して有効なシステム変数の値を表示します。変数にセッション値がない場合は、グローバル値が表示されます。`LOCAL`は`SESSION`の同義語です。</li><li>修飾子が存在しない場合、デフォルトは`SESSION`です。</li></ul> |
| pattern                | LIKE句で変数名と部分一致するために使用するパターン。このパラメータでは、%ワイルドカードを使用できます。 |
| expr                   | WHERE句で変数名`variable_name`あるいは変数値`value`に一致するために使用する式。 |

## 戻り値

| **戻り値**   | **説明**           |
| ------------- | ----------------- |
| Variable_name | 変数の名前。     |
| Value         | 変数の値。       |

## 例

Example 1: LIKE句で変数名を正確に一致させて変数を表示する。

```Plain
mysql> SHOW VARIABLES LIKE 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1行目 (0.01 sec)
```

Example 2: LIKE句とワイルドカード(%)を使用して変数名をおおまかに一致させて変数を表示する。

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
9行目 (0.00 sec)
```

Example 3: WHERE句で変数名を正確に一致させて変数を表示する。

```Plain
mysql> show variables where variable_name = 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1行目 (0.17 sec)
```

Example 4: WHERE句で変数値を正確に一致させて変数を表示する。

```Plain
mysql> show variables where value = '28800';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1行目 (0.70 sec)
```
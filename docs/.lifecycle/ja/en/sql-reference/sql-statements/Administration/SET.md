---
displayed_sidebar: English
---

# SET

## 説明

StarRocksの指定されたシステム変数またはユーザー定義変数を設定します。StarRocksのシステム変数は[SHOW VARIABLES](../Administration/SHOW_VARIABLES.md)を使用して表示できます。システム変数の詳細については[System Variables](../../../reference/System_variable.md)を参照してください。ユーザー定義変数についての詳細は[User-defined variables](../../../reference/user_defined_variables.md)を参照してください。

:::tip

この操作には権限は必要ありません。

:::

## 構文

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## パラメーター

| **パラメーター**          | **説明**                                              |
| ---------------------- | ------------------------------------------------------------ |
| 修飾子:<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>`GLOBAL` 修飾子を使用すると、ステートメントは変数をグローバルに設定します。</li><li>`SESSION` 修飾子を使用すると、ステートメントはセッション内の変数を設定します。`LOCAL` は `SESSION` の同義語です。</li><li>修飾子が存在しない場合、デフォルトは `SESSION` です。</li></ul>グローバル変数とセッション変数の詳細については[System Variables](../../../reference/System_variable.md)を参照してください。<br/>**注記**<br/>ADMIN権限を持つユーザーのみが変数をグローバルに設定できます。 |
| variable_name          | 変数の名前です。                                    |
| value                  | 変数の値です。                                   |

## 例

例 1: セッション内で`time_zone`を`Asia/Shanghai`に設定します。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

例 2: `exec_mem_limit`をグローバルに`2147483648`に設定します。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
Query OK, 0 rows affected (0.00 sec)
```

例 3: 複数のグローバル変数を設定します。`GLOBAL`キーワードは各変数の前に置く必要があります。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

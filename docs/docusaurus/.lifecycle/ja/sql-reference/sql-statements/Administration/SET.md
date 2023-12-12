---
displayed_sidebar: "Japanese"
---

# SET（設定）

## 説明

StarRocksの指定されたシステム変数またはユーザー定義変数を設定します。[SHOW VARIABLES](../Administration/SHOW_VARIABLES.md)を使用して、StarRocksのシステム変数を表示できます。システム変数の詳細については、[System Variables](../../../reference/System_variable.md)を参照してください。ユーザー定義変数の詳細については、[User-defined variables](../../../reference/user_defined_variables.md)を参照してください。

## 構文

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## パラメーター

| **パラメーター**          | **説明**                                              |
| ---------------------- | ------------------------------------------------------------ |
| Modifier:<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>`GLOBAL`修飾子がある場合、ステートメントはグローバルに変数を設定します。</li><li>`SESSION`修飾子がある場合、ステートメントはセッション内に変数を設定します。`LOCAL`は`SESSION`の同義語です。<li>修飾子がない場合、デフォルトは`SESSION`です。</li></ul>グローバルおよびセッション変数の詳細については、[System Variables](../../../reference/System_variable.md)を参照してください。<br/>**注記**<br/>管理者権限を持つユーザーのみが、変数をグローバルに設定できます。 |
| variable_name          | 変数の名前。                                    |
| value                  | 変数の値。                                   |

## 例

例1：`time_zone`をセッション内で`Asia/Shanghai`に設定します。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

例2：`exec_mem_limit`をグローバルに`2147483648`に設定します。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
Query OK, 0 rows affected (0.00 sec)
```

例3：複数のグローバル変数を設定します。各変数には`GLOBAL`キーワードを付ける必要があります。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```
---
displayed_sidebar: "Japanese"
---

# SET（設定）

## 説明

StarRocksに指定されたシステム変数またはユーザー定義の変数を設定します。StarRocksのシステム変数は、[SHOW VARIABLES](../Administration/SHOW_VARIABLES.md)を使用して表示できます。システム変数の詳細については、[システム変数](../../../reference/System_variable.md)を参照してください。ユーザー定義変数の詳細については、[ユーザー定義変数](../../../reference/user_defined_variables.md)を参照してください。

## 構文

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## パラメーター

| **パラメーター**      | **説明**                                                                                  |
| ---------------------- | ------------------------------------------------------------------------------------------- |
| 修飾子:<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>`GLOBAL`修飾子を使用すると、ステートメントはグローバルに変数を設定します。</li><li>`SESSION`修飾子を使用すると、ステートメントはセッション内に変数を設定します。`LOCAL`は`SESSION`の同義語です。</li><li>修飾子がない場合、デフォルトは`SESSION`です。</li></ul>グローバル変数とセッション変数の詳細については、[システム変数](../../../reference/System_variable.md)を参照してください。<br/>**注**<br/>管理者権限を持つユーザーのみが、変数をグローバルに設定できます。 |
| variable_name          | 変数の名前                                                                             |
| value                  | 変数の値                                                                               |

## 例

例1：`time_zone`をセッション内で`Asia/Shanghai`に設定します。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
クエリ OK、0 行が変更されました（0.00 秒）
```

例2：`exec_mem_limit`をグローバルに`2147483648`に設定します。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
クエリ OK、0 行が変更されました（0.00 秒）
```

例3：複数のグローバル変数を設定します。各変数の前に`GLOBAL`キーワードを追加する必要があります。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
クエリ OK、0 行が変更されました（0.00 秒）
```
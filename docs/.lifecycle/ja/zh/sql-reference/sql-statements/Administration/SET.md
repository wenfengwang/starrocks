---
displayed_sidebar: Chinese
---

# SET

## 機能

StarRocksに指定されたシステム変数またはユーザー定義変数を設定します。[SHOW VARIABLES](../Administration/SHOW_VARIABLES.md)を通じてStarRocksのシステム変数を確認できます。システム変数の詳細については、[システム変数](../../../reference/System_variable.md)を参照してください。ユーザー定義変数の詳細については、[ユーザー定義変数](../../../reference/user_defined_variables.md)を参照してください。

:::tip

この操作には権限は必要ありません。

:::

## 文法

```SQL
SET [ GLOBAL | SESSION ] <variable_name> = <value> [, <variable_name> = <value>] ...
```

## パラメータ説明

| **パラメータ**              | **説明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| 修飾子：<ul><li>GLOBAL</li><li>SESSION</li></ul> | <ul><li>`GLOBAL` 修飾子を使用すると、その文は変数の値をグローバル変数の値として設定します。</li><li>`SESSION` 修飾子を使用すると、その文は変数の値を現在の接続にのみ有効として設定します。`LOCAL`を`SESSION`の代わりに使用することもできます。</li><li>修飾子を指定しない場合、デフォルトは `SESSION` です。</li></ul>グローバル変数とセッション変数の詳細については、[システム変数](../../../reference/System_variable.md)を参照してください。<br/>**説明**<br/>SystemレベルのOPERATE権限を持つユーザーのみがグローバル変数を設定できます。 |
| variable_name         | 変数名。                                                     |
| value                 | 変数値。                                                     |

## 例

例1：現在のセッションで `time_zone` を `Asia/Shanghai` に設定します。

```Plain
mysql> SET time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

例2：グローバルに `exec_mem_limit` を `2147483648` に設定します。

```Plain
mysql> SET GLOBAL exec_mem_limit = 2147483648;
Query OK, 0 rows affected (0.00 sec)
```

例3：複数のグローバル変数を同時に設定します。すべての変数名の前に `GLOBAL` キーワードを追加する必要があります。

```Plain
mysql> SET 
       GLOBAL exec_mem_limit = 2147483648,
       GLOBAL time_zone = "Asia/Shanghai";
Query OK, 0 rows affected (0.00 sec)
```

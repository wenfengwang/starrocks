---
displayed_sidebar: Chinese
---

# DROP FUNCTION

## 機能

カスタム関数を削除します。関数の名前と引数の型が完全に一致している必要があります。

このコマンドを実行するユーザーは関数の所有者でなければなりません。

## 文法

```sql
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...])
```

## パラメータ説明

- `GLOBAL`：グローバル関数を削除することを意味します。StarRocksはバージョン3.0から[Global UDF](../../sql-functions/JAVA_UDF.md)の作成をサポートしています。
- `function_name`：削除する関数の名前で、必須です。
- `arg_type`：削除する関数の引数の型で、必須です。

## 例

関数を削除します。

```sql
DROP FUNCTION my_add(INT, INT)
```

## 関連するSQL

- [SHOW FUNCTIONS](./SHOW_FUNCTIONS.md)
- [Java UDF](../../sql-functions/JAVA_UDF.md)

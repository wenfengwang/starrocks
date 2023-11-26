---
displayed_sidebar: "Japanese"
---

# from_binary

## 説明

指定されたバイナリ形式（`binary_type`）に基づいて、バイナリ値をVARCHAR文字列に変換します。次のバイナリ形式がサポートされています：`hex`、`encode64`、および`utf8`。`binary_type`が指定されていない場合、デフォルトは`hex`です。

## 構文

```Haskell
from_binary(binary[, binary_type])
```

## パラメータ

- `binary`：変換する入力バイナリ、必須です。

- `binary_type`：変換のためのバイナリ形式、オプションです。

  - `hex`（デフォルト）：`from_binary`は、入力バイナリをVARCHAR文字列にエンコードするために`hex`メソッドを使用します。
  - `encode64`：`from_binary`は、入力バイナリをVARCHAR文字列にエンコードするために`base64`メソッドを使用します。
  - `utf8`：`from_binary`は、入力バイナリを変換せずにVARCHAR文字列に変換します。

## 戻り値

VARCHAR文字列を返します。

## 例

```Plain
mysql> select from_binary(to_binary('ABAB', 'hex'), 'hex');
+----------------------------------------------+
| from_binary(to_binary('ABAB', 'hex'), 'hex') |
+----------------------------------------------+
| ABAB                                         |
+----------------------------------------------+
1 row in set (0.02 sec)

mysql> select from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64'));
+-----------------------------------------------------------------------------+
| from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64')) |
+-----------------------------------------------------------------------------+
| STARROCKS                                                                   |
+-----------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> select from_binary(to_binary('STARROCKS', 'utf8'), 'utf8');
+-----------------------------------------------------+
| from_binary(to_binary('STARROCKS', 'utf8'), 'utf8') |
+-----------------------------------------------------+
| STARROCKS                                           |
+-----------------------------------------------------+
1 row in set (0.01 sec)

```

## 参考

- [to_binary](to_binary.md)
- [BINARY/VARBINARYデータ型](../../sql-statements/data-types/BINARY.md)

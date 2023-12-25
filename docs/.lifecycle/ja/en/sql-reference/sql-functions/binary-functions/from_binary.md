---
displayed_sidebar: English
---

# from_binary

## 説明

指定されたバイナリ形式 (`binary_type`) に基づいてバイナリ値を VARCHAR 文字列に変換します。サポートされているバイナリ形式には、`hex`、`encode64`、および `utf8` があります。`binary_type` が指定されていない場合、デフォルトは `hex` です。

## 構文

```Haskell
from_binary(binary[, binary_type])
```

## パラメーター

- `binary`: 変換する入力バイナリ。必須です。

- `binary_type`: 変換用のバイナリ形式。オプションです。

  - `hex` (デフォルト): `from_binary` は `hex` メソッドを使用して入力バイナリを VARCHAR 文字列にエンコードします。
  - `encode64`: `from_binary` は `base64` メソッドを使用して入力バイナリを VARCHAR 文字列にエンコードします。
  - `utf8`: `from_binary` は入力バイナリを変換せずに VARCHAR 文字列にします。

## 戻り値

VARCHAR 文字列を返します。

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

## 参照

- [to_binary](to_binary.md)
- [BINARY/VARBINARY データ型](../../sql-statements/data-types/BINARY.md)

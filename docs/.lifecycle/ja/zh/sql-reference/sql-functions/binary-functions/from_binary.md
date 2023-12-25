---
displayed_sidebar: Chinese
---

# from_binary

## 機能

指定されたフォーマットに基づいて、バイナリーデータをVARCHAR型の文字列に変換します。サポートされるバイナリーフォーマットにはHex、Base64、UTF-8が含まれます。指定がない場合、デフォルトはHexです。

この関数はバージョン3.0からサポートされています。

## 構文

```Haskell
from_binary(binary[, binary_type])
```

## パラメータ

- `binary`: 必須、変換されるバイナリ値。
- `binary_type`: オプション、指定されたバイナリフォーマット。有効な値：`hex`、`encode64`、`utf8`。

  - `hex` (デフォルト値): この関数は十六進数エンコーディングを使用してバイナリ値をVARCHAR文字列に変換します。
  - `encode64`: この関数はBase64エンコーディングを使用してバイナリ値をVARCHAR文字列に変換します。
  - `utf8`: この関数はバイナリ値をVARCHAR文字列に直接変換し、変換処理は行いません。

## 戻り値

指定されたフォーマットに基づいて、VARCHAR文字列を返します。

## 例

```Plain
mysql> select from_binary(to_binary('ABAB', 'hex'), 'hex');
+----------------------------------------------+
| from_binary(to_binary('ABAB', 'hex'), 'hex') |
+----------------------------------------------+
| ABAB                                         |
+----------------------------------------------+
1行がセットされました (0.02秒)

mysql> select from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64'));
+-----------------------------------------------------------------------------+
| from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64')) |
+-----------------------------------------------------------------------------+
| STARROCKS                                                                   |
+-----------------------------------------------------------------------------+
1行がセットされました (0.01秒)

mysql> select from_binary(to_binary('STARROCKS', 'utf8'), 'utf8');
+-----------------------------------------------------+
| from_binary(to_binary('STARROCKS', 'utf8'), 'utf8') |
+-----------------------------------------------------+
| STARROCKS                                           |
+-----------------------------------------------------+
1行がセットされました (0.01秒)
```

## 参照

- [to_binary](to_binary.md)
- [BINARY/VARBINARY](../../sql-statements/data-types/BINARY.md)

```markdown
---
displayed_sidebar: "Japanese"
---

# divide

## Description

xをyで割った商を返します。yが0の場合はnullを返します。

## Syntax

```Haskell
divide(x, y)
```

### Parameters

- `x`: サポートされるタイプはDOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128です。

- `y`: `x`と同じサポートされるタイプです。

## Return value

DOUBLEデータ型の値を返します。

## Usage notes

非数値値を指定すると、この関数は`NULL`を返します。

## Examples

```Plain Text
mysql> select divide(3, 2);
+--------------+
| divide(3, 2) |
+--------------+
|          1.5 |
+--------------+
1 row in set (0.00 sec)
```
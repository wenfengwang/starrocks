---
displayed_sidebar: "Japanese"
---

# divide

## 説明

xをyで割った商を返します。yが0の場合、nullを返します。

## 構文

```Haskell
divide(x, y)
```

### パラメータ

- `x`: サポートされる型はDOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128です。

- `y`: `x`と同じサポートされる型です。

## 戻り値

DOUBLEデータ型の値を返します。

## 使用上の注意

数値以外の値を指定した場合、この関数は`NULL`を返します。

## 例

```Plain Text
mysql> select divide(3, 2);
+--------------+
| divide(3, 2) |
+--------------+
|          1.5 |
+--------------+
1 row in set (0.00 sec)
```

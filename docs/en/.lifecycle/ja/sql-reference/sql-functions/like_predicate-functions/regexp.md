---
displayed_sidebar: "Japanese"
---

# 正規表現

## 説明

与えられた式が `pattern` で指定された正規表現に一致するかどうかをチェックします。一致する場合は1が返されます。それ以外の場合は0が返されます。入力パラメータのいずれかがNULLの場合はNULLが返されます。

regexp() は [like()](like.md) よりも複雑な一致条件をサポートしています。

## 構文

```Haskell
BOOLEAN regexp(VARCHAR expr, VARCHAR pattern);
```

## パラメータ

- `expr`: 文字列式です。サポートされるデータ型は VARCHAR です。

- `pattern`: 一致させるパターンです。サポートされるデータ型は VARCHAR です。

## 戻り値

BOOLEAN 値を返します。

## 例

```Plain Text
mysql> select regexp("abc123","abc*");
+--------------------------+
| regexp('abc123', 'abc*') |
+--------------------------+
|                        1 |
+--------------------------+
1 row in set (0.06 sec)

select regexp("abc123","xyz*");
+--------------------------+
| regexp('abc123', 'xyz*') |
+--------------------------+
|                        0 |
+--------------------------+
```

## キーワード

regexp, 正規表現

---
displayed_sidebar: English
---

# 正規表現

## 説明

指定された式が`pattern`で指定された正規表現に一致するかどうかをチェックします。一致する場合は1が返され、一致しない場合は0が返されます。入力パラメータのいずれかがNULLの場合はNULLが返されます。

`regexp()`は[like()](like.md)よりも複雑なマッチング条件をサポートします。

## 構文

```Haskell
BOOLEAN regexp(VARCHAR expr, VARCHAR pattern);
```

## パラメーター

- `expr`: 文字列式。サポートされるデータ型はVARCHARです。

- `pattern`: マッチするパターン。サポートされるデータ型はVARCHARです。

## 戻り値

BOOLEAN値を返します。

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

regexp、regular

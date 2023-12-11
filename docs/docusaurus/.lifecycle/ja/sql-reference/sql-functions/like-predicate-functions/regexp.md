```yaml
---
displayed_sidebar: "English"
---

# regexp

## Description

指定された `pattern` によって定義された正規表現と一致するかどうかをチェックします。一致する場合は、1 が返されます。それ以外の場合は、0 が返されます。入力パラメータのいずれかがNULLの場合は、NULLが返されます。

regexp() は、[like()](like.md)よりも複雑なマッチング条件をサポートしています。

## Syntax

```Haskell
BOOLEAN regexp(VARCHAR expr, VARCHAR pattern);
```

## Parameters

- `expr`: 文字列式。サポートされているデータ型は、VARCHAR です。

- `pattern`: 一致させるパターン。サポートされているデータ型は、VARCHAR です。

## Return value

BOOLEAN 値を返します。

## Examples

```Plain Text
mysql> select regexp("abc123","abc*");
+--------------------------+
| regexp('abc123', 'abc*') |
+--------------------------+
|                        1 |
+--------------------------+
1 行が返されました (0.06 秒)

select regexp("abc123","xyz*");
+--------------------------+
| regexp('abc123', 'xyz*') |
+--------------------------+
|                        0 |
+--------------------------+
```

## Keywords

regexp, 正規表現
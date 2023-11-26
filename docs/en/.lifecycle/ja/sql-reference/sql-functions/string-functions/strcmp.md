---
displayed_sidebar: "Japanese"
---

# strcmp

## 説明

この関数は、2つの文字列を比較します。lhsとrhsが等しい場合は0を返します。lhsがrhsよりも辞書順で前に表示される場合は-1を返します。lhsがrhsよりも辞書順で後に表示される場合は1を返します。引数がNULLの場合、結果はNULLです。

## 構文

```Haskell
INT strcmp(VARCHAR lhs, VARCHAR rhs)
```

## 例

```Plain Text
mysql> select strcmp("test1", "test1");
+--------------------------+
| strcmp('test1', 'test1') |
+--------------------------+
|                        0 |
+--------------------------+

mysql> select strcmp("test1", "test2");
+--------------------------+
| strcmp('test1', 'test2') |
+--------------------------+
|                       -1 |
+--------------------------+

mysql> select strcmp("test2", "test1");
+--------------------------+
| strcmp('test2', 'test1') |
+--------------------------+
|                        1 |
+--------------------------+

mysql> select strcmp("test1", NULL);
+-----------------------+
| strcmp('test1', NULL) |
+-----------------------+
|                  NULL |
+-----------------------+
```

## キーワード

STRCMP

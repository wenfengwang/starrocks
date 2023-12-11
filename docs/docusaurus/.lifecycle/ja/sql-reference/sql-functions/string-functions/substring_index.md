---
displayed_sidebar: "英語"
---

# substring_index

## 説明

この関数は、与えられた文字列をセパレーターで分割してリクエストされた部分を返します。（開始を先頭から数える）

## 構文

```Haskell
VARCHAR substring_index(VARCHAR content, VARCHAR delimiter, INT field)
```

## 例

```Plain Text
select substring_index("hello world", " ", 1);
+----------------------------------------+
| substring_index("hello world", " ", 1) |
+----------------------------------------+
| hello                                  |
+----------------------------------------+
mysql> select substring_index("hello world", " ", 2);
+----------------------------------------+
| substring_index("hello world", " ", 2) |
+----------------------------------------+
| hello world                            |
+----------------------------------------+
mysql> select substring_index("hello world", " ", -1);
+-----------------------------------------+
| substring_index("hello world", " ", -1) |
+-----------------------------------------+
| world                                   |
+-----------------------------------------+
mysql> select substring_index("hello world", " ", -2);
+-----------------------------------------+
| substring_index("hello world", " ", -2) |
+-----------------------------------------+
| hello world                             |
+-----------------------------------------+
mysql> select substring_index("hello world", " ", -3);
+-----------------------------------------+
| substring_index("hello world", " ", -3) |
+-----------------------------------------+
| hello world                             |
+-----------------------------------------+
mysql> select substring_index("hello world", " ", 0);
+----------------------------------------+
| substring_index('hello world', ' ', 0) |
+----------------------------------------+
| NULL                                   |
+----------------------------------------+
```

## キーワード

substring_index
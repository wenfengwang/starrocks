---
displayed_sidebar: "Japanese"
---

# instr

## 説明

この関数は、substr内でstrが最初に現れる位置（1から数えて文字単位）を返します。strがsubstr内で見つからない場合、この関数は0を返します。

## 構文

```Haskell
INT instr(VARCHAR str, VARCHAR substr)
```

## 例

```Plain Text
MySQL > select instr("abc", "b");
+-------------------+
| instr('abc', 'b') |
+-------------------+
|                 2 |
+-------------------+

MySQL > select instr("abc", "d");
+-------------------+
| instr('abc', 'd') |
+-------------------+
|                 0 |
+-------------------+
```

## キーワード

INSTR
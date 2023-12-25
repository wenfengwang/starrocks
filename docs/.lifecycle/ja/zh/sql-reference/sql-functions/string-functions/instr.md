---
displayed_sidebar: Chinese
---

# instr

## 機能

`substr`が`str`内で初めて現れる位置を返します（1から数え始め、文字単位で計算）。`substr`が`str`内に存在しない場合は、0を返します。

## 文法

```Haskell
instr(str, substr)
```

## 引数説明

`str`: 対応するデータ型はVARCHARです。

`substr`: 対応するデータ型はVARCHARです。

## 戻り値の説明

戻り値のデータ型はINTです。

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

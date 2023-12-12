---
displayed_sidebar: "Japanese"
---

# now, current_timestamp, localtime, localtimestamp

## Description

現在の日付と時刻を返します。v3.1以降、結果はマイクロ秒まで正確です。

この関数は、異なるタイムゾーンに対して異なる結果を返す場合があります。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。

## Syntax

```Haskell
DATETIME NOW()
```

## Examples

```Plain Text
MySQL > select now();
+---------------------+
| now()               |
+---------------------+
| 2019-05-27 15:58:25 |
+---------------------+

-- v3.1以降、結果はマイクロ秒まで正確です。
MySQL > select now();
+----------------------------+
| now()                      |
+----------------------------+
| 2023-11-18 12:54:34.878000 |
+----------------------------+
```

## keyword

NOW, now
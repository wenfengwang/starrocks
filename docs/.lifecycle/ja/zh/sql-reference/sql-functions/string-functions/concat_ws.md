---
displayed_sidebar: Chinese
---

# concat_ws

## 機能

区切り文字を使用して、2つ以上の文字列を新しい文字列に連結します。新しい文字列は区切り文字で接続されます。

### 文法

```Haskell
VARCHAR concat_ws(VARCHAR sep, VARCHAR str,...)
```

### パラメータ説明

- `sep`: 区切り文字、データタイプは VARCHAR。
- `str`: 連結される文字列、データタイプは VARCHAR。この関数は空の文字列をスキップせず、NULL 値はスキップします。

### 戻り値の説明

VARCHAR タイプの文字列を返します。区切り文字が NULL の場合、NULL を返します。

### 例

例1：区切り文字として `r` を使用し、`starrocks` を返します。

```Plain Text
MySQL > select concat_ws("r", "sta", "rocks");
+--------------------------------+
| concat_ws('r', 'sta', 'rocks') |
+--------------------------------+
| starrocks                      |
+--------------------------------+
```

例2：区切り文字として `NULL` を使用し、NULL を返します。

```Plain Text
MySQL > select concat_ws(NULL, "star", "rocks");
+----------------------------------+
| concat_ws(NULL, 'star', 'rocks') |
+----------------------------------+
| NULL                             |
+----------------------------------+
```

例3：区切り文字として `r` を使用し、NULL 値をスキップします。

```Plain Text
MySQL > select concat_ws("r", "sta", NULL,"rocks");
+-------------------------------------+
| concat_ws("r", "sta", NULL,"rocks") |
+-------------------------------------+
| starrocks                           |
+-------------------------------------+
```

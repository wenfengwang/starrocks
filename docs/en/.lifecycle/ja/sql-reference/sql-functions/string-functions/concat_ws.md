---
displayed_sidebar: "Japanese"
---

# concat_ws

## 説明

この関数は、最初の引数 sep を区切り文字として使用し、2番目以降の引数を結合して文字列を形成します。区切り文字がNULLの場合、結果もNULLになります。concat_wsは空の文字列をスキップしませんが、NULLの値はスキップします。

## 構文

```Haskell
VARCHAR concat_ws(VARCHAR sep, VARCHAR str,...)
```

## 例

```Plain Text
MySQL > select concat_ws("or", "d", "is");
+----------------------------+
| concat_ws('or', 'd', 'is') |
+----------------------------+
| starrocks                      |
+----------------------------+

MySQL > select concat_ws(NULL, "d", "is");
+----------------------------+
| concat_ws(NULL, 'd', 'is') |
+----------------------------+
| NULL                       |
+----------------------------+

MySQL > select concat_ws("or", "d", NULL,"is");
+---------------------------------+
| concat_ws("or", "d", NULL,"is") |
+---------------------------------+
| starrocks                           |
+---------------------------------+
```

## キーワード

CONCAT_WS,CONCAT,WS

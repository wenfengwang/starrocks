---
displayed_sidebar: "Japanese"
---

# last_query_id

## 説明

現在のセッションで最近実行されたクエリのIDを取得します。

## 構文

```Haskell
VARCHAR last_query_id();
```

## パラメーター

なし

## 戻り値

VARCHAR型の値を返します。

## 例

```Plain Text
mysql> select last_query_id();
+--------------------------------------+
| last_query_id()                      |
+--------------------------------------+
| 7c1d8d68-bbec-11ec-af65-00163e1e238f |
+--------------------------------------+
1 行が返されました (0.00 秒)
```

## キーワード

LAST_QUERY_ID
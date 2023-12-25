---
displayed_sidebar: Chinese
---

# last_query_id

## 機能

最後に実行されたクエリのIDを返します。

## 文法

```Haskell
VARCHAR last_query_id();
```

## パラメータ説明

なし。

## 戻り値の説明

VARCHAR型の値を返します。

## 例

```Plain Text
mysql> select last_query_id();
+--------------------------------------+
| last_query_id()                      |
+--------------------------------------+
| 7c1d8d68-bbec-11ec-af65-00163e1e238f |
+--------------------------------------+
1行がセットされました (0.00秒)
```

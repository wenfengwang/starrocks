---
displayed_sidebar: English
---

# host_name

## 説明

計算が実行されるノードのホスト名を取得します。

## 構文

```Haskell
host_name();
```

## パラメーター

なし

## 戻り値

VARCHAR 型の値を返します。

## 例

```Plaintext
select host_name();
+-------------+
| host_name() |
+-------------+
| sandbox-sql |
+-------------+
1 行がセットされました (0.01 秒)
```

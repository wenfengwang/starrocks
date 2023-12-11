---
displayed_sidebar: "Japanese"
---

# host_name

## 説明

計算が実行されているノードのホスト名を取得します。

## 構文

```Haskell
host_name();
```

## パラメーター

なし

## 戻り値

VARCHAR値を返します。

## 例

```Plaintext
select host_name();
+-------------+
| host_name() |
+-------------+
| sandbox-sql |
+-------------+
1 行が選択されました (0.01 秒)
```
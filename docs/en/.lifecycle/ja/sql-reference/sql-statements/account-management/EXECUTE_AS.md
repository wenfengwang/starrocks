---
displayed_sidebar: "Japanese"
---

# EXECUTE AS（実行ユーザーの切り替え）

## 説明

ユーザーの権限を取得した後、EXECUTE AS ステートメントを使用して、現在のセッションの実行コンテキストを切り替えることができます。

このコマンドはv2.4からサポートされています。

## 構文

```SQL
EXECUTE AS user WITH NO REVERT
```

## パラメーター

`user`: 事前に存在するユーザーです。

## 使用上の注意

- EXECUTE AS ステートメントを呼び出す現在のログインユーザーは、他のユーザーを模倣する権限を付与されている必要があります。詳細については、[GRANT](../account-management/GRANT.md)を参照してください。
- EXECUTE AS ステートメントには、WITH NO REVERT句を含める必要があります。これは、現在のセッションが終了する前に、現在のセッションの実行コンテキストを元のログインユーザーに戻すことができないことを意味します。

## 例

現在のセッションの実行コンテキストをユーザー `test2` に切り替えます。

```SQL
EXECUTE AS test2 WITH NO REVERT;
```

切り替えに成功した後、`select current_user()` コマンドを実行して現在のユーザーを取得できます。

```SQL
select current_user();
+-----------------------------+
| CURRENT_USER()              |
+-----------------------------+
| 'default_cluster:test2'@'%' |
+-----------------------------+
```

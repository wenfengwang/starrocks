---
displayed_sidebar: English
---

# EXECUTE AS

## 説明

ユーザーを偽装する権限を取得した後、EXECUTE AS ステートメントを使用して現在のセッションの実行コンテキストをそのユーザーに切り替えることができます。

このコマンドは v2.4 からサポートされています。

## 構文

```SQL
EXECUTE AS user WITH NO REVERT
```

## パラメーター

`user`: このユーザーは既に存在している必要があります。

## 使用上の注意

- EXECUTE AS ステートメントを呼び出す現在のログインユーザーは、他のユーザーを偽装する権限を付与されている必要があります。詳細については、[GRANT](../account-management/GRANT.md)を参照してください。
- EXECUTE AS ステートメントには WITH NO REVERT 句が含まれている必要があります。これは、現在のセッションが終了するまで、実行コンテキストを元のログインユーザーに戻すことができないことを意味します。

## 例

現在のセッションの実行コンテキストをユーザー `test2` に切り替えます。

```SQL
EXECUTE AS test2 WITH NO REVERT;
```

切り替えに成功すると、`select current_user()` コマンドを実行して現在のユーザーを取得できます。

```SQL
select current_user();
+-----------------------------+
| CURRENT_USER()              |
+-----------------------------+
| 'default_cluster:test2'@'%' |
+-----------------------------+
```

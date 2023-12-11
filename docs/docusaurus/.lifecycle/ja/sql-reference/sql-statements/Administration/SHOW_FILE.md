```yaml
---
displayed_sidebar: "Japanese"
---

# ファイルの表示

データベースに保存されているファイルに関する情報を表示するには、SHOW FILE ステートメントを実行できます。

## 構文

```SQL
SHOW FILE [FROM database]
```

このステートメントによって返されるファイル情報は次のとおりです:

- `FileId`: ファイルのグローバルに一意なID。

- `DbName`: ファイルが属するデータベース。

- `Catalog`: ファイルが属するカテゴリ。

- `FileName`: ファイルの名前。

- `FileSize`: ファイルのサイズ。単位はバイトです。

- `MD5`: ファイルを確認するために使用されるメッセージダイジェストアルゴリズム。

## 例

`my_database` に保存されているファイルを表示します。

```SQL
SHOW FILE FROM my_database;
```
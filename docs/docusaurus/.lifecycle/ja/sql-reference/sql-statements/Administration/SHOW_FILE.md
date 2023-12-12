---
displayed_sidebar: "Japanese"
---

# ファイルを表示

データベースに格納されているファイルに関する情報を表示するために、SHOW FILE ステートメントを実行できます。

## 構文

```SQL
SHOW FILE [FROM database]
```

このステートメントによって返されるファイル情報は以下の通りです：

- `FileId`: ファイルのグローバルに一意なID。

- `DbName`: ファイルが所属するデータベース。

- `Catalog`: ファイルが所属するカテゴリ。

- `FileName`: ファイルの名前。

- `FileSize`: ファイルのサイズ。単位はバイト。

- `MD5`: ファイルをチェックするために使用されるメッセージダイジェストアルゴリズム。

## 例

`my_database` に格納されているファイルを表示します。

```SQL
SHOW FILE FROM my_database;
```
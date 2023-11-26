---
displayed_sidebar: "Japanese"
---

# ファイルの表示

SHOW FILE ステートメントを実行すると、データベースに保存されているファイルの情報を表示することができます。

## 構文

```SQL
SHOW FILE [FROM database]
```

このステートメントによって返されるファイル情報は以下の通りです：

- `FileId`：ファイルのグローバルに一意なIDです。

- `DbName`：ファイルが所属するデータベースです。

- `Catalog`：ファイルが所属するカテゴリです。

- `FileName`：ファイルの名前です。

- `FileSize`：ファイルのサイズです。単位はバイトです。

- `MD5`：ファイルをチェックするために使用されるメッセージダイジェストアルゴリズムです。

## 例

`my_database` に保存されているファイルを表示します。

```SQL
SHOW FILE FROM my_database;
```

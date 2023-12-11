---
displayed_sidebar: "Japanese"
---

# リポジトリを表示

## 説明

StarRocks で作成されたリポジトリを表示します。

## 構文

```SQL
SHOW REPOSITORIES
```

## 返り値

| **返り値** | **説明**                                         |
| ---------- | ----------------------------------------------- |
| RepoId     | リポジトリID。                                    |
| RepoName   | リポジトリ名。                                    |
| CreateTime | リポジトリの作成時間。                            |
| IsReadOnly | リポジトリが読み取り専用かどうか。                 |
| Location   | リモートストレージシステム内のリポジトリの場所。 |
| Broker     | リポジトリを作成するために使用されるブローカー。 |
| ErrMsg     | リポジトリへの接続中のエラーメッセージ。           |

## 例

例 1: StarRocks で作成されたリポジトリを表示します。

```SQL
SHOW REPOSITORIES;
```
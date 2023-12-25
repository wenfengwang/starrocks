---
displayed_sidebar: Chinese
---

# リポジトリの表示

## 機能

現在作成されているリポジトリを表示します。

## 文法

```SQL
SHOW REPOSITORIES
```

## 戻り値

| **項目**   | **説明**                |
| ---------- | ----------------------- |
| RepoId     | リポジトリのID。        |
| RepoName   | リポジトリ名。          |
| CreateTime | リポジトリの作成時間。  |
| IsReadOnly | リポジトリが読み取り専用かどうか。 |
| Location   | リモートストレージシステムのパス。 |
| Broker     | リポジトリを作成するためのBroker。 |
| ErrMsg     | リポジトリの接続エラーメッセージ。 |

## 例

例1：作成されたリポジトリを表示します。

```SQL
SHOW REPOSITORIES;
```

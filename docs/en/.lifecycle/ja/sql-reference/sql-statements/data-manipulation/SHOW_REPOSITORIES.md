---
displayed_sidebar: "Japanese"
---

# リポジトリの表示

## 説明

StarRocksで作成されたリポジトリを表示します。

## 構文

```SQL
SHOW REPOSITORIES
```

## 戻り値

| **戻り値**   | **説明**                                                 |
| ------------ | -------------------------------------------------------- |
| RepoId       | リポジトリのID。                                          |
| RepoName     | リポジトリの名前。                                        |
| CreateTime   | リポジトリの作成時刻。                                    |
| IsReadOnly   | リポジトリが読み取り専用かどうか。                          |
| Location     | リモートストレージシステム内のリポジトリの場所。           |
| Broker       | リポジトリの作成に使用されるブローカー。                    |
| ErrMsg       | リポジトリへの接続中に発生したエラーメッセージ。             |

## 例

例1: StarRocksで作成されたリポジトリを表示します。

```SQL
SHOW REPOSITORIES;
```

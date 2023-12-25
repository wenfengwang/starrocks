---
displayed_sidebar: English
---

# リポジトリの表示

## 説明

StarRocksで作成されたリポジトリを表示します。

## 構文

```SQL
SHOW REPOSITORIES
```

## 戻り値

| **戻り値** | **説明**                                          |
| ---------- | -------------------------------------------------------- |
| RepoId     | リポジトリID。                                           |
| RepoName   | リポジトリ名。                                         |
| CreateTime | リポジトリの作成時間。                                |
| IsReadOnly | リポジトリが読み取り専用かどうか。                          |
| Location   | リモートストレージシステムでのリポジトリの位置。 |
| Broker     | リポジトリを作成するために使用されたBroker。                    |
| ErrMsg     | リポジトリへの接続時のエラーメッセージ。       |

## 例

例1: StarRocksで作成されたリポジトリを表示します。

```SQL
SHOW REPOSITORIES;
```

```yaml
---
displayed_sidebar: "Japanese"
---

# リポジトリの表示

## 説明

StarRocks で作成されたリポジトリを表示します。

## 構文

```SQL
SHOW REPOSITORIES
```

## 戻り値

| **戻り値** | **説明**                                                     |
| ---------- | ------------------------------------------------------------ |
| RepoId     | リポジトリID。                                                |
| RepoName   | リポジトリ名。                                                |
| CreateTime | リポジトリ作成時間。                                          |
| IsReadOnly | リポジトリが読み取り専用かどうか。                              |
| Location   | リモートストレージシステム内のリポジトリの場所。            |
| Broker     | リポジトリの作成に使用されたブローカー。                      |
| ErrMsg     | リポジトリへの接続中のエラーメッセージ。                        |

## 例

例1：StarRocks で作成されたリポジトリを表示します。

```SQL
SHOW REPOSITORIES;
```
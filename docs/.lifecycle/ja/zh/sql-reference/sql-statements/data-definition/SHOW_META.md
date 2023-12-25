---
displayed_sidebar: Chinese
---

# SHOW META

## 機能

統計情報のメタデータを表示します。このコマンドは、基本統計情報とヒストグラム統計情報のメタデータを表示することをサポートしています。このステートメントはバージョン2.4からサポートされています。

CBO統計情報の収集についての詳細は、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。

### 基本統計情報のメタデータ

#### 構文

```SQL
SHOW STATS META [WHERE predicate]
```

このステートメントは以下の列を返します。

| **列名**   | **説明**                                            |
| ---------- | --------------------------------------------------- |
| Database   | データベース名。                                    |
| Table      | テーブル名。                                        |
| Columns    | 列名。                                              |
| Type       | 統計情報のタイプ。`FULL`は全量、`SAMPLE`はサンプルを意味します。 |
| UpdateTime | 現在のテーブルの最新統計情報更新時間。              |
| Properties | カスタムパラメータ情報。                            |
| Healthy    | 統計情報の健全性。                                  |

### ヒストグラム統計情報のメタデータ

#### 構文

```SQL
SHOW HISTOGRAM META [WHERE predicate];
```

このステートメントは以下の列を返します。

| **列名**   | **説明**                                  |
| ---------- | ----------------------------------------- |
| Database   | データベース名。                          |
| Table      | テーブル名。                              |
| Column     | 列名。                                    |
| Type       | 統計情報のタイプ。ヒストグラムは `HISTOGRAM`として固定。 |
| UpdateTime | 現在のテーブルの最新統計情報更新時間。    |
| Properties | カスタムパラメータ情報。                  |

## 関連文書

CBO統計情報の収集についてもっと知りたい場合は、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。

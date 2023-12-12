---
displayed_sidebar: "Japanese"
---

# ALTER TABLEの取り消し

## 説明

指定されたテーブルに対してALTER TABLE文で実行された以下の操作を取り消します。

- テーブルスキーマ: 列の追加および削除、列の並び替え、および列のデータ型の変更。
- ロールアップインデックス: ロールアップインデックスの作成および削除。

この文は同期操作であり、テーブルに対する `ALTER_PRIV` 権限が必要です。

## 構文

- スキーマ変更の取り消し。

    ```SQL
    CANCEL ALTER TABLE COLUMN FROM [db_name.]table_name
    ```

- ロールアップインデックスの変更の取り消し。

    ```SQL
    CANCEL ALTER TABLE ROLLUP FROM [db_name.]table_name
    ```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | No           | テーブルが属するデータベースの名前。このパラメータを指定しない場合、デフォルトで現在のデータベースが使用されます。 |
| table_name    | Yes          | テーブルの名前。                                              |

## 例

例1: `example_db`データベース内の `example_table` に対するスキーマ変更を取り消す。

```SQL
CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
```

例2: 現在のデータベース内の `example_table` に対するロールアップインデックスの変更を取り消す。

```SQL
CANCEL ALTER TABLE ROLLUP FROM example_table;
```
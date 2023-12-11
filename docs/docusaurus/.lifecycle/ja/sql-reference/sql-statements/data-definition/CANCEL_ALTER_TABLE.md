---
displayed_sidebar: "Japanese"
---

# ALTER TABLEのキャンセル

## 説明

指定されたテーブルに対してALTER TABLEステートメントで実行された以下の操作をキャンセルします。

- テーブルスキーマ：列の追加および削除、列の並び替え、および列のデータ型の変更。
- ロールアップインデックス：ロールアップインデックスの作成および削除。

このステートメントは同期的な操作であり、テーブルに対して`ALTER_PRIV`権限を持っている必要があります。

## 構文

- スキーマ変更のキャンセル。

    ```SQL
    CANCEL ALTER TABLE COLUMN FROM [db_name.]table_name
    ```

- ロールアップインデックスの変更のキャンセル。

    ```SQL
    CANCEL ALTER TABLE ROLLUP FROM [db_name.]table_name
    ```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | いいえ           | テーブルが所属するデータベースの名前。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。 |
| table_name    | はい          | テーブルの名前。                                              |

## 例

例1：`example_db`データベースの`example_table`に対するスキーマの変更をキャンセルする。

```SQL
CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
```

例2：現在のデータベースで`example_table`に対するロールアップインデックスの変更をキャンセルする。

```SQL
CANCEL ALTER TABLE ROLLUP FROM example_table;
```
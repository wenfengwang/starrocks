---
displayed_sidebar: "Japanese"
---

# ALTER TABLE のキャンセル

## 説明

指定されたテーブルに対して ALTER TABLE ステートメントで実行される以下の操作をキャンセルします：

- テーブルスキーマ：列の追加と削除、列の並び替え、列のデータ型の変更。
- ロールアップインデックス：ロールアップインデックスの作成と削除。

このステートメントは同期操作であり、テーブルに対して `ALTER_PRIV` 権限が必要です。

## 構文

- スキーマの変更をキャンセルします。

    ```SQL
    CANCEL ALTER TABLE COLUMN FROM [db_name.]table_name
    ```

- ロールアップインデックスの変更をキャンセルします。

    ```SQL
    CANCEL ALTER TABLE ROLLUP FROM [db_name.]table_name
    ```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| ------------- | ---------- | ------------------------------------------------------------ |
| db_name       | いいえ       | テーブルが所属するデータベースの名前。このパラメータが指定されていない場合、現在のデータベースがデフォルトで使用されます。 |
| table_name    | はい         | テーブルの名前。                                              |

## 例

例1：`example_db` データベースの `example_table` に対するスキーマの変更をキャンセルします。

```SQL
CANCEL ALTER TABLE COLUMN FROM example_db.example_table;
```

例2：現在のデータベースの `example_table` に対するロールアップインデックスの変更をキャンセルします。

```SQL
CANCEL ALTER TABLE ROLLUP FROM example_table;
```

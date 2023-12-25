---
displayed_sidebar: Chinese
---

# エクスポートのキャンセル

## 機能

指定されたデータエクスポートジョブをキャンセルします。`CANCELLED` または `FINISHED` 状態のエクスポートジョブはキャンセルできません。CANCEL EXPORT は非同期操作であり、実行後は [SHOW EXPORT](./SHOW_EXPORT.md) ステートメントを使用してキャンセルが成功したかどうかを確認できます。状態 (`State`) が `CANCELLED` の場合、エクスポートジョブのキャンセルに成功したことを意味します。

> **注意**
>
> この操作を実行するには、対象テーブルの EXPORT 権限が必要です。

## 语法

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | いいえ   | データベース名。指定しない場合、現在のデータベースの指定されたエクスポートジョブがキャンセルされます。 |
| query_id       | はい     | エクスポートジョブのクエリ ID。[LAST_QUERY_ID()](../../sql-functions/utility-functions/last_query_id.md) 関数を使用して取得できます。この関数を使用すると、最後のクエリ ID のみが取得できることに注意してください。 |

## 例

例1：現在のデータベースで、クエリ ID が `921d8f80-7c9d-11eb-9342-acde48001121` のエクスポートジョブをキャンセルします。

```SQL
CANCEL EXPORT
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001121";
```

例2：データベース `example_db` で、クエリ ID が `921d8f80-7c9d-11eb-9342-acde48001122` のエクスポートジョブをキャンセルします。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```

---
displayed_sidebar: English
---

# エクスポートのキャンセル

## 説明

指定されたデータアンロードジョブをキャンセルします。`CANCELLED` または `FINISHED` 状態のアンロードジョブはキャンセルできません。アンロードジョブのキャンセルは非同期プロセスです。[SHOW EXPORT](../data-manipulation/SHOW_EXPORT.md) ステートメントを使用して、アンロードジョブが正常にキャンセルされたかどうかを確認できます。`State` の値が `CANCELLED` であれば、アンロードジョブは正常にキャンセルされています。

CANCEL EXPORT ステートメントを使用するには、指定されたアンロードジョブが属するデータベースに対して、以下のいずれかの権限を少なくとも一つ持っている必要があります：`SELECT_PRIV`, `LOAD_PRIV`, `ALTER_PRIV`, `CREATE_PRIV`, `DROP_PRIV`, `USAGE_PRIV`。権限の詳細については、[GRANT](../account-management/GRANT.md)を参照してください。

## 構文

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## パラメーター

| **パラメーター** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | いいえ           | アンロードジョブが属するデータベースの名前。このパラメーターが指定されていない場合、現在のデータベース内のアンロードジョブがキャンセルされます。 |
| query_id      | はい          | アンロードジョブのクエリID。このIDは [LAST_QUERY_ID()](../../sql-functions/utility-functions/last_query_id.md) 関数を使用して取得できます。この関数は最新のクエリIDのみを返すことに注意してください。 |

## 例

例 1: 現在のデータベースでクエリIDが `921d8f80-7c9d-11eb-9342-acde48001121` のアンロードジョブをキャンセルします。

```SQL
CANCEL EXPORT
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001121";
```

例 2: `example_db` データベースでクエリIDが `921d8f80-7c9d-11eb-9342-acde48001121` のアンロードジョブをキャンセルします。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```

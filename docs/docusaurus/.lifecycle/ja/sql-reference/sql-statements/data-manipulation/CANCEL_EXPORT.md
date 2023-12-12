```yaml
---
displayed_sidebar: "Japanese"
---

# エクスポートのキャンセル

## 説明

指定されたデータのアンローディングジョブをキャンセルします。`CANCELLED`または`FINISHED`の状態のアンローディングジョブはキャンセルできません。アンローディングジョブのキャンセルは非同期プロセスです。[SHOW EXPORT](../data-manipulation/SHOW_EXPORT.md)ステートメントを使用して、アンローディングジョブが正常にキャンセルされたかどうかを確認できます。`State`の値が`CANCELLED`であれば、アンローディングジョブは正常にキャンセルされています。

CANCEL EXPORTステートメントは、指定されたアンローディングジョブが属するデータベースに対して少なくとも次の特権を持っている必要があります：`SELECT_PRIV`、`LOAD_PRIV`、`ALTER_PRIV`、`CREATE_PRIV`、`DROP_PRIV`、`USAGE_PRIV`。特権の説明についての詳細は、[GRANT](../account-management/GRANT.md)を参照してください。

## 構文

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | いいえ           | アンローディングジョブが属するデータベースの名前。このパラメータが指定されていない場合、現在のデータベースのアンローディングジョブがキャンセルされます。 |
| query_id      | はい          | アンローディングジョブのクエリID。[LAST_QUERY_ID()](../../sql-functions/utility-functions/last_query_id.md)関数を使用してIDを取得できます。ただし、この関数は最新のクエリIDのみを返します。 |

## 例

例1：クエリIDが`921d8f80-7c9d-11eb-9342-acde48001121`のアンローディングジョブを現在のデータベースでキャンセルします。

```SQL
CANCEL EXPORT
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001121";
```

例2：クエリIDが`921d8f80-7c9d-11eb-9342-acde48001121`のアンローディングジョブを`example_db`データベースでキャンセルします。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```
```
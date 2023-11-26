---
displayed_sidebar: "Japanese"
---

# エクスポートのキャンセル

## 説明

指定されたデータのアンロードジョブをキャンセルします。`CANCELLED`または`FINISHED`の状態のアンロードジョブはキャンセルできません。アンロードジョブのキャンセルは非同期プロセスです。アンロードジョブが正常にキャンセルされたかどうかは、[SHOW EXPORT](../data-manipulation/SHOW_EXPORT.md)ステートメントを使用して確認できます。`State`の値が`CANCELLED`であれば、アンロードジョブは正常にキャンセルされています。

CANCEL EXPORTステートメントを使用するには、指定されたアンロードジョブが所属するデータベースに対して、次のいずれかの権限を持っている必要があります：`SELECT_PRIV`、`LOAD_PRIV`、`ALTER_PRIV`、`CREATE_PRIV`、`DROP_PRIV`、および`USAGE_PRIV`。権限の詳細については、[GRANT](../account-management/GRANT.md)を参照してください。

## 構文

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | いいえ   | アンロードジョブが所属するデータベースの名前です。このパラメータが指定されていない場合、現在のデータベースのアンロードジョブがキャンセルされます。 |
| query_id       | はい     | アンロードジョブのクエリIDです。[LAST_QUERY_ID()](../../sql-functions/utility-functions/last_query_id.md)関数を使用してIDを取得できます。ただし、この関数は最新のクエリIDのみを返します。 |

## 例

例1：クエリIDが`921d8f80-7c9d-11eb-9342-acde48001121`のアンロードジョブを現在のデータベースでキャンセルする場合。

```SQL
CANCEL EXPORT
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001121";
```

例2：クエリIDが`921d8f80-7c9d-11eb-9342-acde48001121`のアンロードジョブを`example_db`データベースでキャンセルする場合。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```

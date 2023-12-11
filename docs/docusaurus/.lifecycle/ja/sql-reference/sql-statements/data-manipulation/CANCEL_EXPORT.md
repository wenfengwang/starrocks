---
displayed_sidebar: "Japanese"
---

# エクスポートのキャンセル

## 説明

指定されたデータのアンローディングジョブをキャンセルします。 `CANCELLED`または` FINISHED`の状態のアンローディングジョブはキャンセルできません。 アンローディングジョブのキャンセルは非同期プロセスです。 [SHOW EXPORT](../data-manipulation/SHOW_EXPORT.md)ステートメントを使用して、アンローディングジョブが正常にキャンセルされたかどうかを確認できます。 `State`の値が` CANCELLED`である場合、アンローディングジョブは正常にキャンセルされます。

CANCEL EXPORT ステートメントを実行するには、指定されたアンローディングジョブが所属するデータベースに対して、次のいずれかの特権が少なくとも1つ必要です：`SELECT_PRIV`、`LOAD_PRIV`、`ALTER_PRIV`、`CREATE_PRIV`、`DROP_PRIV`、`USAGE_PRIV`。特権の説明についての詳細は、[GRANT](../account-management/GRANT.md)を参照してください。

## 構文

```SQL
CANCEL EXPORT
[FROM db_name]
WHERE QUERYID = "query_id"
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | いいえ           | アンローディングジョブが所属するデータベースの名前。このパラメータを指定しない場合、現在のデータベースのアンローディングジョブがキャンセルされます。 |
| query_id      | はい          | アンローディングジョブのクエリID。[LAST_QUERY_ID()](../../sql-functions/utility-functions/last_query_id.md)関数を使用してIDを取得できます。ただし、この関数は最新のクエリIDのみを返します。 |

## 例

**例1：** 現在のデータベースでクエリIDが`921d8f80-7c9d-11eb-9342-acde48001121`であるアンローディングジョブをキャンセルする。

```SQL
CANCEL EXPORT
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001121";
```

**例2：** `example_db`データベースでクエリIDが`921d8f80-7c9d-11eb-9342-acde48001121`であるアンローディングジョブをキャンセルする。

```SQL
CANCEL EXPORT 
FROM example_db 
WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```
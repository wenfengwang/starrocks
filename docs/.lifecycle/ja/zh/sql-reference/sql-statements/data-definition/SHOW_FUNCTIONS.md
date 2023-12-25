---
displayed_sidebar: Chinese
---

# 関数の表示

## 機能

データベースに存在するすべてのカスタム（システム提供）関数を表示します。データベースが指定されていない場合は、現在のセッションが存在するデータベースを直接照会します。

## 文法

```sql
SHOW [FULL] [BUILTIN] [GLOBAL] FUNCTIONS [IN|FROM <db_name>] [LIKE 'function_pattern']
```

## パラメータ説明

* `FULL`: 関数の詳細情報を表示します。
* `BUILTIN`: システムが提供する関数を表示します。
* `GLOBAL`: グローバル関数を表示します。StarRocks はバージョン 3.0 から [Global UDF](../../sql-functions/JAVA_UDF.md) の作成をサポートしています。
* `db_name`: 照会するデータベースの名前。
* `function_pattern`: 関数名をフィルタリングするために使用します。

## 例

```sql
-- \Gは結果を列ごとに表示することを意味します
mysql> show full functions in testDb\G
*************************** 1. row ***************************
        Signature: my_add(INT,INT)
      Return Type: INT
    Function Type: Scalar
Intermediate Type: NULL
       Properties: {"symbol":"_ZN9starrocks_udf6AddUdfEPNS_15FunctionContextERKNS_6IntValES4_","object_file":"http://host:port/libudfsample.so","md5":"cfe7a362d10f3aaf6c49974ee0f1f878"}
*************************** 2. row ***************************
        Signature: my_count(BIGINT)
      Return Type: BIGINT
    Function Type: Aggregate
Intermediate Type: NULL
       Properties: {"object_file":"http://host:port/libudasample.so","finalize_fn":"_ZN9starrocks_udf13CountFinalizeEPNS_15FunctionContextERKNS_9BigIntValE","init_fn":"_ZN9starrocks_udf9CountInitEPNS_15FunctionContextEPNS_9BigIntValE","merge_fn":"_ZN9starrocks_udf10CountMergeEPNS_15FunctionContextERKNS_9BigIntValEPS2_","md5":"37d185f80f95569e2676da3d5b5b9d2f","update_fn":"_ZN9starrocks_udf11CountUpdateEPNS_15FunctionContextERKNS_6IntValEPNS_9BigIntValE"}

2 rows in set (0.00 sec)

mysql> show builtin functions in testDb like 'year%';
+---------------+
| Function Name |
+---------------+
| year          |
| years_add     |
| years_diff    |
| years_sub     |
+---------------+
2 rows in set (0.00 sec)
```

## 関連するSQL

* [DROP FUNCTION](./DROP_FUNCTION.md)
* [Java UDF](../../sql-functions/JAVA_UDF.md)

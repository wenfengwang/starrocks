---
displayed_sidebar: "Japanese"
---

# 関数の表示

## 説明

データベース内のすべてのカスタム（または組み込み）関数をクエリします。データベースが指定されていない場合、デフォルトで現在のデータベースが使用されます。

## 構文

```sql
SHOW [FULL] [BUILTIN] FUNCTIONS [IN|FROM db] [LIKE 'function_pattern']
```

## パラメータ

```plain text
full: すべての関数を表示することを示します。

builtin: システムによって提供される関数を表示することを示します。

db: クエリするデータベースの名前です。

function_pattern: 関数名をフィルタリングするために使用されるパターンです。
```

## 例

```Plain Text
mysql> show full functions in testDb\G
*************************** 1. 行 ***************************
        シグネチャ: my_add(INT,INT)
      戻り値の型: INT
    関数の種類: Scalar
中間型: NULL
       プロパティ: {"symbol":"_ZN9starrocks_udf6AddUdfEPNS_15FunctionContextERKNS_6IntValES4_","object_file":"http://host:port/libudfsample.so","md5":"cfe7a362d10f3aaf6c49974ee0f1f878"}
*************************** 2. 行 ***************************
        シグネチャ: my_count(BIGINT)
      戻り値の型: BIGINT
    関数の種類: Aggregate
中間型: NULL
       プロパティ: {"object_file":"http://host:port/libudasample.so","finalize_fn":"_ZN9starrocks_udf13CountFinalizeEPNS_15FunctionContextERKNS_9BigIntValE","init_fn":"_ZN9starrocks_udf9CountInitEPNS_15FunctionContextEPNS_9BigIntValE","merge_fn":"_ZN9starrocks_udf10CountMergeEPNS_15FunctionContextERKNS_9BigIntValEPS2_","md5":"37d185f80f95569e2676da3d5b5b9d2f","update_fn":"_ZN9starrocks_udf11CountUpdateEPNS_15FunctionContextERKNS_6IntValEPNS_9BigIntValE"}

2 行が返されました (0.00 秒)
mysql> show builtin functions in testDb like 'year%';
+---------------+
| 関数名        |
+---------------+
| year          |
| years_add     |
| years_diff    |
| years_sub     |
+---------------+
2 行が返されました (0.00 秒)
```

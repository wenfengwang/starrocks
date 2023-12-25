---
displayed_sidebar: Chinese
---

# キャスト

## 機能

CAST 関数を使用して、JSON 型のデータと SQL 型のデータを相互に変換します。

## 文法

- JSON 型のデータを SQL 型に変換します。

```sql
CAST(json_expr AS sql_data_type)
```

- SQL 型のデータを JSON 型に変換します。

```sql
CAST(sql_expr AS JSON)
```

## パラメータ説明

- `json_expr`：SQL 型に変換する JSON 式。
- `sql_data_type`：変換後の SQL 型で、文字列型（STRING、VARCHAR、CHAR を含む）、BOOLEAN、数値型（TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT を含む）があります。
- `sql_expr`：JSON 型に変換する SQL 式。サポートされるデータ型は `sql_data_type` と同じです。

## 戻り値の説明

- `CAST(json_expr AS sql_data_type)` を使用すると、指定された `sql_data_type` の型の値が返されます。
- `CAST(sql_expr AS JSON)` を使用すると、JSON 型の値が返されます。

## 注意事項

SQL 型のデータを JSON 型に変換する場合：

- 数値型の値が JSON のサポートする精度を超える場合、数値オーバーフローを避けるために SQL 型の NULL が返されます。

- SQL 型の NULL は、JSON 型の NULL に変換されず、SQL 型の NULL のままです。

JSON 型のデータを SQL 型に変換する場合：

- 互換性のある型の変換がサポートされており、例えば JSON 文字列を SQL 文字列に変換できます。

- 型変換が互換性がない場合、例えば JSON の数値を SQL 文字列に変換しようとすると、NULL が返されます。

- 数値型の変換でオーバーフローが発生した場合、SQL 型の NULL が返されます。

- JSON 型の NULL を SQL 型に変換すると、SQL の NULL が返されます。

- JSON 型のデータを VARCHAR 型に変換する場合、変換前が JSON string 型であれば、引用符なしの VARCHAR 型のデータが返されます。

## 例

例1：JSON 型のデータを SQL 型に変換します。

```Plain Text
-- JSON を INT に変換します。
mysql> select cast(parse_json('{"a": 1}') -> 'a' as int);
+--------------------------------------------+
| CAST((parse_json('{"a": 1}')->'a') AS INT) |
+--------------------------------------------+
|                                          1 |
+--------------------------------------------+

-- JSON 文字列を VARCHAR に変換します。
mysql> select cast(parse_json('"star"') as varchar);
+---------------------------------------+
| CAST(parse_json('"star"') AS VARCHAR) |
+---------------------------------------+
| star                                  |
+---------------------------------------+

-- JSON オブジェクトを VARCHAR に変換します。
mysql> select cast(parse_json('{"star": 1}') as varchar);
+--------------------------------------------+
| CAST(parse_json('{"star": 1}') AS VARCHAR) |
+--------------------------------------------+
| {"star": 1}                                |
+--------------------------------------------+

-- JSON 配列を VARCHAR に変換します。
mysql> select cast(parse_json('[1,2,3]') as varchar);
+----------------------------------------+
| CAST(parse_json('[1,2,3]') AS VARCHAR) |
+----------------------------------------+
| [1, 2, 3]                              |
+----------------------------------------+
```

例2：SQL 型のデータを JSON 型に変換します。

```Plain Text
-- INT を JSON に変換します。
mysql> select cast(1 as json);
+-----------------+
| CAST(1 AS JSON) |
+-----------------+
| 1               |
+-----------------+

-- VARCHAR を JSON に変換します。
mysql> select cast("star" as json);
+----------------------+
| CAST('star' AS JSON) |
+----------------------+
| "star"               |
+----------------------+

-- BOOLEAN を JSON に変換します。
mysql> select cast(true as json);
+--------------------+
| CAST(TRUE AS JSON) |
+--------------------+
| true               |
+--------------------+
```

---
displayed_sidebar: "Japanese"
---

# cast（キャスト）

## 説明

JSONタイプとSQLタイプの間の値を変換します。

## 構文

- JSONからSQLへの変換

```Haskell
cast(json_expr AS sql_data_type)
```

- SQLからJSONへの変換

```Haskell
cast(sql_expr AS JSON)
```

## パラメータ

- `json_expr`：JSON値をSQL値に変換する式。
  
- `sql_data_type`：JSON値を変換するSQLデータ型。STRING、VARCHAR、CHAR、BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、およびFLOATデータ型のみがサポートされています。

- `sql_expr`：JSON値をSQL値に変換する式。このパラメータは、`sql_data_type`パラメータでサポートされているすべてのSQLデータ型をサポートしています。

## 戻り値

- `cast(json_expr AS sql_data_type)`構文を使用すると、キャスト関数は、`sql_data_type`パラメータで指定されたSQLデータ型の値を返します。

- `cast(sql_expr AS JSON)`構文を使用すると、キャスト関数はJSON値を返します。

## 使用上の注意

- SQLからJSONへの変換

  - SQL値がJSONでサポートされている精度を超える場合、算術オーバーフローを防ぐため、キャスト関数は`NULL`を返します。

  - SQL値が`NULL`の場合、キャスト関数はSQL値`NULL`をJSON値`NULL`に変換しません。返り値は引き続きSQL値`NULL`です。

- JSONからSQLへの変換

  - キャスト関数は、互換性のあるJSONデータ型とSQLデータ型間のみの変換をサポートしています。たとえば、JSON文字列をSQL文字列に変換することができます。

  - キャスト関数は、互換性のないJSONデータ型とSQLデータ型間の変換をサポートしていません。たとえば、JSON数値をSQL文字列に変換すると、関数は`NULL`を返します。

  - 算術オーバーフローが発生すると、キャスト関数はSQL値`NULL`を返します。

  - JSON値`NULL`をSQL値に変換すると、関数はSQL値`NULL`を返します。

  - JSON文字列をVARCHAR値に変換すると、関数は二重引用符（"）で囲まれていないVARCHAR値を返します。

## 例

例1：JSON値をSQL値に変換する。

```plaintext
-- JSON値をINT値に変換する。
mysql> select cast(parse_json('{"a": 1}') -> 'a' as int);
+--------------------------------------------+
| CAST((parse_json('{"a": 1}')->'a') AS INT) |
+--------------------------------------------+
|                                          1 |
+--------------------------------------------+

-- JSON文字列をVARCHAR値に変換する。
mysql> select cast(parse_json('"star"') as varchar);
+---------------------------------------+
| cast(parse_json('"star"') AS VARCHAR) |
+---------------------------------------+
| star                                  |
+---------------------------------------+

-- JSONオブジェクトをVARCHAR値に変換する。
mysql> select cast(parse_json('{"star": 1}') as varchar);
+--------------------------------------------+
| cast(parse_json('{"star": 1}') AS VARCHAR) |
+--------------------------------------------+
| {"star": 1}                                |
+--------------------------------------------+

-- JSON配列をVARCHAR値に変換する。
mysql> select cast(parse_json('[1,2,3]') as varchar);
+----------------------------------------+
| cast(parse_json('[1,2,3]') AS VARCHAR) |
+----------------------------------------+
| [1, 2, 3]                              |
+----------------------------------------+
```

例2：SQL値をJSON値に変換する。

```plaintext
-- INT値をJSON値に変換する。
mysql> select cast(1 as json);
+-----------------+
| cast(1 AS JSON) |
+-----------------+
| 1               |
+-----------------+

-- VARCHAR値をJSON値に変換する。
mysql> select cast("star" as json);
+----------------------+
| cast('star' AS JSON) |
+----------------------+
| "star"               |
+----------------------+

-- BOOLEAN値をJSON値に変換する。
mysql> select cast(true as json);
+--------------------+
| cast(TRUE AS JSON) |
+--------------------+
| true               |
+--------------------+
```
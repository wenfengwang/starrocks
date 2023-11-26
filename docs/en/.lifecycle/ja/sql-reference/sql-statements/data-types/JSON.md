---
displayed_sidebar: "Japanese"
---

# JSON

StarRocksはv2.2.0以降、JSONデータ型をサポートしています。このトピックでは、JSONの基本的な概念、JSON列の作成方法、JSONデータのロード方法、JSONデータのクエリ方法、およびJSONデータの構築と処理に使用するJSON関数と演算子について説明します。

## JSONとは

JSONは、半構造化データ向けに設計された軽量なデータ交換形式です。JSONはデータを階層的なツリー構造で表現し、データの格納や分析において柔軟で読み書きしやすいです。JSONは`NULL`値と以下のデータ型をサポートしています：NUMBER、STRING、BOOLEAN、ARRAY、OBJECT。

JSONに関する詳細な情報については、[JSONのウェブサイト](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)を参照してください。JSONの入力および出力構文に関する情報については、[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)のJSON仕様を参照してください。

StarRocksはJSONデータのストレージと効率的なクエリと分析を両方サポートしています。StarRocksは入力テキストを直接格納せず、パースのコストを削減しクエリの効率を向上させるためにJSONデータをバイナリ形式で格納します。

## JSONデータの使用

### JSON列の作成

テーブルを作成する際に、`JSON`キーワードを使用して`j`列をJSON列として指定することができます。

```sql
CREATE TABLE `tj` (
    `id` INT(11) NOT NULL COMMENT "",
    `j`  JSON NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
);
```

### データのロードとJSONデータとしての保存

StarRocksでは、次の方法でデータをロードし、データをJSONデータとして保存することができます：

- 方法1：`INSERT INTO`を使用してテーブルのJSON列にデータを書き込む方法。以下の例では、`tj`という名前のテーブルを使用し、テーブルの`j`列がJSON列です。

```plaintext
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> parse_json関数はSTRINGデータをJSONデータとして解釈することができます。json_object関数はJSONオブジェクトを構築するか、既存のテーブルをJSONファイルに変換することができます。詳細については、[parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md)および[json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md)を参照してください。

- 方法2：Stream Loadを使用してJSONファイルをロードし、ファイルをJSONデータとして保存する方法。詳細については、[JSONデータのロード](../../../loading/StreamLoad.md#load-json-data)を参照してください。

  - ルートのJSONオブジェクトをロードする場合は、`jsonpaths`を`$`に設定します。
  - JSONオブジェクトの特定の値をロードする場合は、`jsonpaths`を`$.a`に設定します。ここで、`a`はキーを指定します。StarRocksでサポートされているJSONパス式の詳細については、[JSONパス](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path-expressions)を参照してください。

- 方法3：Broker Loadを使用してParquetファイルをロードし、ファイルをJSONデータとして保存する方法。詳細については、[Broker Load](../data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksはParquetファイルのロード時に以下のデータ型変換をサポートしています。

| Parquetファイルのデータ型                                  | JSONデータ型 |
| -------------------------------------------------------- | ------------ |
| INTEGER (INT8、INT16、INT32、INT64、UINT8、UINT16、UINT32、およびUINT64) | NUMBER       |
| FLOATおよびDOUBLE                                          | NUMBER       |
| BOOLEAN                                                  | BOOLEAN      |
| STRING                                                   | STRING       |
| MAP                                                      | OBJECT       |
| STRUCT                                                   | OBJECT       |
| LIST                                                     | ARRAY        |
| UNIONおよびTIMESTAMP                                      | サポートされていません |

- 方法4：[Routine](../../../loading/RoutineLoad.md)ロードを使用してKafkaからStarRocksにJSONデータを連続的にロードします。

### JSONデータのクエリと処理

StarRocksは、JSONデータのクエリと処理、およびJSON関数と演算子の使用をサポートしています。

以下の例では、`tj`という名前のテーブルを使用し、テーブルの`j`列をJSON列として指定しています。

```plaintext
mysql> select * from tj;
+------+----------------------+
| id   |          j           |
+------+----------------------+
| 1    | {"a": 1, "b": true}  |
| 2    | {"a": 2, "b": false} |
| 3    | {"a": 3, "b": true}  |
| 4    | {"a": 4, "b": false} |
+------+----------------------+
```

例1：JSON列のデータをフィルタリングして、`id=1`のフィルタ条件を満たすデータを取得します。

```plaintext
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

例2：JSON列`j`のデータをフィルタリングして、指定されたフィルタ条件を満たすデータを取得します。

> `j->'a'`はJSONデータを返します。データを比較するためには、最初の例を使用することができます（この例では暗黙の型変換が行われます）。または、CAST関数を使用してJSONデータをINTに変換し、データを比較することもできます。

```plaintext
mysql> select * from tj where j->'a' = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |


mysql> select * from tj where cast(j->'a' as INT) = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |
+------+---------------------+
```

例3：CAST関数を使用してテーブルのJSON列の値をBOOLEAN値に変換し、指定されたフィルタ条件を満たすデータをフィルタリングします。

```plaintext
mysql> select * from tj where cast(j->'b' as boolean);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

例4：CAST関数を使用してテーブルのJSON列の値をBOOLEAN値に変換し、指定されたフィルタ条件を満たすデータをフィルタリングし、データに算術演算を行います。

```plaintext
mysql> select cast(j->'a' as int) from tj where cast(j->'b' as boolean);
+-----------------------+
|  CAST(j->'a' AS INT)  |
+-----------------------+
|          3            |
|          1            |
+-----------------------+

mysql> select sum(cast(j->'a' as int)) from tj where cast(j->'b' as boolean);
+----------------------------+
| sum(CAST(j->'a' AS INT))  |
+----------------------------+
|              4             |
+----------------------------+
```

例5：JSON列をソートキーとして使用してテーブルのデータをソートします。

```plaintext
mysql> select * from tj
    ->        where j->'a' <= parse_json('3')
    ->        order by cast(j->'a' as int);
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    1 | {"a": 1, "b": true}  |
|    2 | {"a": 2, "b": false} |
|    3 | {"a": 3, "b": true}  |
|    4 | {"a": 4, "b": false} |
+------+----------------------+
4 rows in set (0.05 sec)
```

## JSON関数と演算子

JSON関数と演算子を使用してJSONデータを構築および処理することができます。詳細については、[JSON関数と演算子の概要](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md)を参照してください。

## 制限事項と使用上の注意

- JSON値の最大長は16 MBです。

- ORDER BY、GROUP BY、およびJOIN句はJSON列への参照をサポートしていません。JSON列への参照を作成する場合は、参照を作成する前にJSON列をSQL列に変換するためにCAST関数を使用してください。詳細については、[cast](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md)を参照してください。

- JSON列はDuplicate Key、Primary Key、およびUnique Keyテーブルでサポートされています。Aggregateテーブルではサポートされていません。

- JSON列は、DUPLICATE KEY、PRIMARY KEY、およびUNIQUE KEYテーブルのパーティションキー、バケットキー、または次元列として使用することはできません。ORDER BY、GROUP BY、およびJOIN句では使用できません。

- StarRocksでは、次のJSON比較演算子を使用してJSONデータをクエリすることができます：`<`、`<=`、`>`、`>=`、`=`、および`!=`。`IN`を使用してJSONデータをクエリすることはできません。

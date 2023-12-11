---
displayed_sidebar: "Japanese"
---

# JSON（JSON）

StarRocksは、v2.2.0以降、JSONデータ型をサポートするようになりました。このトピックでは、JSONの基本的な概念について説明します。JSON列の作成、JSONデータのロード、JSONデータのクエリ、およびJSONデータの構築と処理におけるJSON関数と演算子の使用についても説明します。

## JSONとは

JSONは、半構造化データ向けに設計された軽量でデータ交換のためのフォーマットです。JSONはデータを柔軟で幅広い範囲のデータストレージおよび分析シナリオで読み書きしやすい階層構造で表現します。JSONは`NULL`値と次のデータ型をサポートしています：NUMBER、STRING、BOOLEAN、ARRAY、およびOBJECT。

JSONに関する詳細情報については、[JSON websiten](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)をご覧ください。また、JSONの入力および出力構文に関する情報は、[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)のJSON仕様を参照してください。

StarRocksは、JSONデータの蓄積および効率的なクエリと分析の両方をサポートしています。StarRocksは、入力テキストを直接格納するのではなく、JSONデータをバイナリ形式で格納し、解析コストを削減し、クエリの効率を向上させています。

## JSONデータの使用

### JSON列の作成

テーブルを作成する際、`JSON`キーワードを使用して、`j`列をJSON列として指定できます。

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

### データをロードし、JSONデータとしてデータを保存

StarRocksでは、データをロードし、JSONデータとして保存するための以下の方法を提供しています：

- 方法1：`INSERT INTO`を使用して、テーブルのJSON列にデータを書き込みます。次の例では、`tj`という名前のテーブルが使用され、テーブルの`j`列はJSON列です。

```plaintext
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> parse_json関数は、STRINGデータをJSONデータとして解釈します。json_object関数は、JSONオブジェクトを構築するか、既存のテーブルをJSONファイルに変換します。詳細については、[parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md)および[json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md)を参照してください。

- 方法2：Stream Loadを使用して、JSONファイルをロードし、ファイルをJSONデータとして保存します。詳細については、[JSONデータのロード](../../../loading/StreamLoad.md#load-json-data)を参照してください。

  - ルートのJSONオブジェクトをロードする場合は、`jsonpaths`を`$`に設定します。
  - 特定のJSONオブジェクトの値をロードする場合は、`jsonpaths`を`$.a`に設定します。ここで、`a`はキーを指定します。StarRocksでサポートされているJSONパス式に関する詳細については、「[JSON path](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path-expressions)」を参照してください。

- 方法3：Broker Loadを使用して、Parquetファイルをロードし、ファイルをJSONデータとして保存します。詳細については、[Broker Load](../data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksは、Parquetファイルのロード時に以下のデータ型変換をサポートしています。

| Parquetファイルのデータ型                             | JSONデータ型 |
| ---------------------------------------------------- | ------------ |
| INTEGER（INT8、INT16、INT32、INT64、UINT8、UINT16、UINT32、およびUINT64） | NUMBER       |
| FLOATおよびDOUBLE                                      | NUMBER       |
| BOOLEAN                                               | BOOLEAN      |
| STRING                                                | STRING       |
| MAP                                                   | OBJECT       |
| STRUCT                                                | OBJECT       |
| LIST                                                  | ARRAY        |
| UNIONやTIMESTAMPなどのその他のデータ型                 | サポートされていません |

- 方法4：[Routine](../../../loading/RoutineLoad.md) Loadを使用して、KafkaからStarRocksに連続してJSONデータをロードします。

### JSONデータのクエリと処理

StarRocksは、JSONデータのクエリと処理をサポートし、JSON関数と演算子の使用をサポートしています。

次の例では、`tj`という名前のテーブルが使用され、テーブルの`j`列がJSON列として指定されています。

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

Example 1: JSON列のデータをフィルタリングして、`id=1`のフィルタ条件に一致するデータを取得する。

```plaintext
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

Example 2: JSON列`j`のデータをフィルタリングして、指定されたフィルタ条件に一致するデータを取得する。

> `j->'a'`はJSONデータを返します。ここではデータを比較するために最初の例を使用できます（この例では暗黙の変換が行われます）。または、CAST関数を使用してJSONデータをINTに変換してデータを比較することができます。

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

Example 3: CAST関数を使用してテーブルのJSON列の値をBOOLEAN値に変換し、指定されたフィルタ条件に一致するデータをフィルタリングする。

```plaintext
mysql> select * from tj where cast(j->'b' as boolean);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

Example 4: CAST関数を使用してテーブルのJSON列の値をBOOLEAN値に変換し、指定されたフィルタ条件に一致するデータをフィルタリングし、データに算術演算を行う。

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

Example 5: JSON列をソートキーとして使用してテーブルのデータをソートする。

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

JSON関数と演算子を使用して、JSONデータの構築と処理を行うことができます。詳細については、「[JSON関数と演算子の概要](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md)」を参照してください。

## 制限事項と使用上の注意

- JSON値の最大長は16 MBです。

- ORDER BY、GROUP BY、およびJOIN句では、JSON列への参照がサポートされていません。JSON列への参照を作成する場合は、参照を作成する前にJSON列をSQL列に変換するためにCAST関数を使用してください。詳細については、[cast](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md)を参照してください。

- JSON列は、Duplicate Key、Primary Key、Unique Keyテーブルでサポートされています。集計テーブルではサポートされていません。

- JSON列は、DUPLICATE KEY、PRIMARY KEY、UNIQUE KEYテーブルのパーティションキー、バケツキー、または次元列として使用することはできません。ORDER BY、GROUP BY、JOIN句でも使用することはできません。

- StarRocksでは、次のJSON比較演算子を使用してJSONデータをクエリすることができます：`<`, `<=`, `>`, `>=`, `=`, `!=`。`IN`を使用してJSONデータをクエリすることはできません。
---
displayed_sidebar: "Japanese"
---

# JSON（JSON）

StarRocks は v2.2.0 から JSON データ型をサポートするようになりました。このトピックでは JSON の基本的なコンセプトについて説明します。また、JSON 列の作成方法、JSON データのロード、JSON データのクエリ、および JSON データの構築と処理に使用する JSON 関数および演算子について説明します。

## JSON とは

JSON は、半構造化データ用に設計された軽量なデータ交換形式です。JSON は、柔軟であるため、さまざまなデータストレージおよび分析シナリオでデータを階層的なツリー構造で表示し、読み書きしやすくしています。JSON は `NULL` 値と次のデータ型をサポートしています: NUMBER、STRING、BOOLEAN、ARRAY、および OBJECT。

JSON に関する詳細情報については、[JSON ウェブサイト](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)を参照してください。JSON の入力および出力構文に関する情報については、[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4) の JSON 仕様をご覧ください。

StarRocks は、JSON データの格納と効率的なクエリと分析をサポートしています。StarRocks は入力テキストを直接格納せず、JSON データをバイナリ形式で格納して解析コストを削減し、クエリ効率を向上させます。

## JSON データの使用

### JSON 列の作成

テーブルを作成する際、`JSON` キーワードを使用して `j` 列を JSON 列として指定できます。

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

### JSON データのロードおよびデータの JSON データとしての保存

StarRocks では、次の方法でデータのロードおよびデータの JSON データとしての保存ができます:

- 方法 1: `INSERT INTO` を使用してテーブルの JSON 列にデータを書き込むことができます。次の例では、`tj` という名前のテーブルが使用され、テーブルの `j` 列が JSON 列です。

```plaintext
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> parse_json 関数は、STRING データを JSON データとして解釈できます。json_object 関数は、JSON オブジェクトを構築するか、既存のテーブルを JSON ファイルに変換することができます。詳細については、[parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md) および [json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md) を参照してください。

- 方法 2: Stream Load を使用して JSON ファイルをロードし、ファイルを JSON データとして保存することができます。詳細については、[JSON データのロード](../../../loading/StreamLoad.md#load-json-data) を参照してください。

  - ルート JSON オブジェクトをロードする場合は、 `jsonpaths` を `$` に設定します。
  - JSON オブジェクトの特定の値をロードする場合は、 `jsonpaths` を `$.a` に設定します。ここで `a` はキーを指定します。StarRocks でサポートされている JSON パス式についての詳細については、[JSON path](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path-expressions) を参照してください。

- 方法 3: Broker Load を使用して Parquet ファイルをロードし、ファイルを JSON データとして保存することができます。詳細については、[Broker Load](../data-manipulation/BROKER_LOAD.md) を参照してください。

StarRocks では、Parquet ファイルの読み込み時に次のデータ型の変換をサポートしています。

| Parquet ファイルのデータ型                                  | JSON データ型  |
| ------------------------------------------------------------ | -------------- |
| INTEGER (INT8, INT16, INT32, INT64, UINT8, UINT16, UINT32 および UINT64) | NUMBER         |
| FLOAT および DOUBLE                                          | NUMBER         |
| BOOLEAN                                                      | BOOLEAN        |
| STRING                                                       | STRING         |
| MAP                                                          | OBJECT         |
| STRUCT                                                       | OBJECT         |
| LIST                                                         | ARRAY          |
| UNION および TIMESTAMP などのその他のデータ型             | サポートされていません |

- 方法 4: [Routine](../../../loading/RoutineLoad.md) load を使用して、Kafka から StarRocks に JSON データを継続的にロードすることができます。

### JSON データのクエリと処理

StarRocks では、JSON データのクエリと処理、および JSON 関数および演算子の使用がサポートされています。

次の例では、`tj` という名前のテーブルが使用され、テーブルの `j` 列が JSON 列として指定されています。

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

例 1: JSON 列のデータをフィルタして、`id=1` のフィルタ条件を満たすデータを取得します。

```plaintext
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

例 2: JSON 列 `j` のデータをフィルタして、指定されたフィルタ条件を満たすデータを取得します。

> `j->'a'` は JSON データを返します。この例ではデータを比較できます（この例では暗黙の変換が行われます）。別の方法として、JSON データを INT に変換し、データを比較する際に CAST 関数を使用することができます。

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

例 3: テーブルの JSON 列の値を BOOLEAN 値に変換するために CAST 関数を使用します。その後、指定されたフィルタ条件を満たす JSON 列のデータをフィルタします。

```plaintext
mysql> select * from tj where cast(j->'b' as boolean);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

例 4: テーブルの JSON 列の値を BOOLEAN 値に変換するために CAST 関数を使用します。その後、指定されたフィルタ条件を満たす JSON 列のデータをフィルタし、データに算術演算を実行します。

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

例 5: JSON 列をソートキーとしてテーブルのデータをソートします。

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

## JSON 関数と演算子

JSON 関数および演算子を使用して、JSON データの構築および処理を行うことができます。詳細については、[JSON 関数および演算子の概要](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md) を参照してください。

## 制限および使用上の注意事項

- JSON 値の最大長は 16 MB です。
- ORDER BY、GROUP BY、JOIN 句では、JSON カラムの参照はサポートされていません。JSON カラムへの参照を作成する場合は、参照を作成する前に JSON カラムを SQL カラムに変換するために CAST 関数を使用してください。詳細については、[cast](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md) を参照してください。

- JSON カラムは、Duplicate Key、Primary Key、Unique Key テーブルでサポートされていますが、Aggregate テーブルではサポートされていません。

- JSON カラムは、Duplicate Key、Primary Key、Unique Key テーブルのパーティションキー、バケットキー、次元カラムとして使用することはできません。また、ORDER BY、GROUP BY、JOIN 句でも使用できません。

- StarRocks では、次の JSON 比較演算子を使用して JSON データのクエリを実行できます: `<`、 `<=`、 `>`、 `>=`、 `=`、 `!=`。ただし、`IN` を使用して JSON データをクエリすることはできません。
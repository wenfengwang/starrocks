---
displayed_sidebar: Chinese
---

# JSON

バージョン 2.2.0 から、StarRocks は JSON をサポートしています。この文書では、JSON の基本概念と、StarRocks で JSON 型の列を作成し、JSON データをインポートしてクエリする方法、そして JSON 関数と演算子を使用して JSON データを構築および処理する方法について説明します。

## JSON とは

JSON は軽量なデータ交換フォーマットであり、JSON 型のデータは半構造化データで、ツリー構造をサポートしています。JSON データは階層が明確で、柔軟な構造を持ち、読みやすく処理しやすいため、データの保存や分析シーンで広く使用されています。JSON がサポートするデータ型には、数値型（NUMBER）、文字列型（STRING）、ブーリアン型（BOOLEAN）、配列型（ARRAY）、オブジェクト型（OBJECT）、および NULL 値があります。

JSON の詳細については、[JSON 公式サイト](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)をご覧ください。JSON データの入力と出力の文法については、JSON 仕様 [RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4) を参照してください。

StarRocks は JSON データの保存と効率的なクエリ分析をサポートしています。StarRocks は、入力されたテキストを直接保存するのではなく、JSON データをバイナリ形式でエンコードして保存するため、データの計算クエリ時に解析コストを削減し、クエリの効率を向上させます。

## JSON データの使用

### JSON 型の列の作成

テーブルを作成する際に、キーワード `JSON` を使用して、列 `j` を JSON 型として指定します。

```SQL
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

### JSON 型としてデータをインポートして保存

StarRocks は以下の方法でデータをインポートし、JSON 型として保存することをサポートしています。

- 方法 1：`INSERT INTO` を使用して、JSON 型の列（例えば列 `j`）にデータを書き込みます。

```SQL
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> PARSE_JSON 関数は、文字列型のデータを基に JSON 型のデータを構築することができます。JSON_OBJECT 関数は JSON オブジェクト型のデータを構築することができ、既存のテーブルを JSON 型に変換することができます。詳細は [PARSE_JSON](../../sql-functions/json-functions/json-constructor-functions/parse_json.md) と [JSON_OBJECT](../../sql-functions/json-functions/json-constructor-functions/json_object.md) を参照してください。

- 方法 2：Stream Load を使用して JSON ファイルをインポートし、JSON 型として保存します。インポート方法については、[JSON データのインポート](../../../loading/StreamLoad.md) を参照してください。
  - JSON ファイルのルートノードの JSON オブジェクトを JSON 型としてインポートして保存するには、`jsonpaths` を `$` に設定します。
  - JSON ファイル内のある JSON オブジェクトの値（value）を JSON 型としてインポートして保存するには、`jsonpaths` を `$.a`（a はキーを表します）に設定します。JSON パス式の詳細については、[JSON path](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path) を参照してください。
  
- 方法 3：Broker Load を使用して Parquet ファイルをインポートし、JSON 型として保存します。インポート方法については、[Broker Load](../data-manipulation/BROKER_LOAD.md) を参照してください。

インポート時にサポートされるデータ型の変換は以下の通りです：

| Parquet ファイル内のデータ型                                     | 変換後の JSON データ型 |
| ------------------------------------------------------------ | -------------------- |
| 整数型（INT8、INT16、INT32、INT64、UINT8、UINT16、UINT32、UINT64） | JSON 数値型          |
| 浮動小数点型（FLOAT、DOUBLE）                                    | JSON 数値型          |
| BOOLEAN                                                      | JSON ブーリアン型          |
| STRING                                                       | JSON 文字列型        |
| MAP                                                          | JSON オブジェクト型          |
| STRUCT                                                       | JSON オブジェクト型          |
| LIST                                                         | JSON 配列型          |
| UNION、TIMESTAMP などその他の型                                  | サポートされていません             |

- 方法 4：[Routine Load](../../../loading/RoutineLoad.md#导入-json-数据) を使用して Kafka の JSON 形式のデータを継続的に消費し、StarRocks にインポートします。

### JSON 型のデータのクエリと処理

StarRocks は JSON 型のデータのクエリと処理をサポートし、JSON 関数と演算子の使用もサポートしています。

以下の例は、テーブル `tj` を使用して説明します。

```Plain Text
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

例 1：フィルタ条件 `id=1` に基づいて、JSON 型の列で条件を満たすデータを選択します。

```Plain Text
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

例 2：JSON 型の列を基にフィルタリングし、条件を満たすデータをテーブルから選択します。

> 次の例では `j->'a'` は JSON 型のデータを返します。データに対して暗黙の型変換が行われている最初の例と比較することができます。または、CAST 関数を使用して JSON 型のデータを INT に変換し、比較することもできます。

```Plain Text
mysql> select * from tj where j->'a' = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |
+------+---------------------+

mysql> select * from tj where cast(j->'a' as INT) = 1; 
+------+---------------------+
|   id |         j           |
+------+---------------------+
|   1  | {"a": 1, "b": true} |
+------+---------------------+
```

例 3：JSON 型の列を基にフィルタリングし（CAST 関数を使用して JSON 型の列を BOOLEAN 型に変換することができます）、条件を満たすデータをテーブルから選択します。

```Plain Text
mysql> select * from tj where cast(j->'b' as boolean);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

例 4：JSON 型の列を基にフィルタリングし（CAST 関数を使用して JSON 型の列を BOOLEAN 型に変換することができます）、条件を満たす JSON 型の列のデータを選択し、数値演算を行います。

```Plain Text
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

例 5：JSON 型の列を基にソートします。

```Plain Text
mysql> select * from tj
       where j->'a' <= parse_json('3')
       order by cast(j->'a' as int);
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    1 | {"a": 1, "b": true}  |
|    2 | {"a": 2, "b": false} |
|    3 | {"a": 3, "b": true}  |
|    4 | {"a": 4, "b": false} |
+------+----------------------+
```

## JSON 関数と演算子

JSON 関数と演算子は、JSON データの構築と処理に使用できます。詳細は、[JSON 関数と演算子](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md) を参照してください。

## 制限事項と注意点

- 現在、JSON 型のデータがサポートする最大長は 16 MB です。
- ORDER BY、GROUP BY、JOIN 句では JSON 型の列を参照することはできません。参照が必要な場合は、CAST 関数を使用して JSON 型の列を他の SQL 型に変換することができます。具体的な変換方法については、[JSON 型の変換](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md) を参照してください。
- JSON 型の列は、明細モデル、主キーモデル、更新モデルのテーブルに存在することができますが、集約モデルのテーブルには存在することができません。
- JSON 型の列を明細モデル、主キーモデル、更新モデルのテーブルのパーティションキー、バケットキー、次元列として使用することはできません。また、JOIN、GROUP BY、ORDER BY 句での使用もサポートされていません。
- StarRocks は `<`、`<=`、`>`、`>=`、`=`、`!=` の演算子を使用した JSON データのクエリをサポートしていますが、IN 演算子を使用したクエリはサポートしていません。

---
displayed_sidebar: Chinese
---

# MAP

## 説明

MAP は複雑なデータ型で、順不同のキーと値のペア（key-value pair）を格納するために使用されます。例えば `{a:1, b:2, c:3}` のようになります。Map の中の Key は重複することができません。Map は最大で 14 レベルのネストをサポートします。

StarRocks はバージョン 3.1 から MAP 型のデータの保存とインポートをサポートしています。テーブルを作成する際に MAP 列を定義し、MAP データをテーブルにインポートし、MAP データをクエリすることができます。

StarRocks はバージョン 2.5 から、データレイク内の複雑なデータ型 MAP と STRUCT のクエリをサポートしています。StarRocks が提供する External Catalog を使用して、Apache Hive™、Apache Hudi、Apache Iceberg 内の MAP と STRUCT データをクエリすることができます。ORC および Parquet 形式のファイルのクエリのみをサポートしています。

External Catalog を使用して外部データソースをクエリする方法については、[Catalog 概要](../../../data_source/catalog/catalog_overview.md) および対応する Catalog ドキュメントを参照してください。

## 構文

```SQL
MAP<key_type,value_type>
```

- `key_type`：Key のデータ型。StarRocks がサポートする基本データ型（Primitive Type）でなければなりません。例えば数値、文字列、日付型などです。複雑な型はサポートされていません。例えば HLL、JSON、ARRAY、MAP、BITMAP、STRUCT などです。
- `value_type`：Value のデータ型。StarRocks がサポートする任意の型、複雑な型を含むことができます。

Key と Value の値は NULL にすることができます。

## MAP 型の列の定義

CREATE TABLE ステートメントで MAP 型の列を定義し、その後で MAP データをその列にインポートすることができます。

```SQL
-- 単純な map を定義。
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- ネストされた map を定義。
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- NULL でない map を定義。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

MAP 列には以下の使用制限があります：

- テーブルの Key 列としては使用できず、Value 列としてのみ使用可能です。
- テーブルのパーティション列（PARTITION BY で定義される列）としては使用できません。
- テーブルのバケット列（DISTRIBUTED BY で定義される列）としては使用できません。

## SQL での MAP の構築

以下の二つの構文を使用して Map を構築できます：

- `map{key_expr:value_expr, ...}`：Map の要素はコンマ（`,`）で区切られ、要素内の Key と Value はコロン（`:`）で区切られます。例えば `map{a:1, b:2, c:3}` のようになります。
- `map(key_expr, value_expr ...)`：Key と Value の式はペアでなければならず、そうでない場合は構築に失敗します。例えば `map(a,1,b,2,c,3)` のようになります。

StarRocks は入力された Key と Value から Key と Value のデータ型を推測することができます。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- {1:1,2:2,3:3} を返します。
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- {1:"apple",2:"orange",3:"pear"} を返します。
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- {true:{3.13:"abc"},false:{}} を返します。
```

もし Key または Value のデータ型が一致しない場合、StarRocks は適切なデータ型（supertype）を自動的に推測します。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- {1.0:2.2,1.2:21.0} を返します。
select map{12:"a", "100":1, NULL:NULL} as string_string; -- {"12":"a","100":"1",null:null} を返します。
```

Map を構築する際に `<>` を使用して map のデータ型を定義することもできます。入力された Key と Value は定義された型に変換可能でなければなりません。

```SQL
select map<FLOAT,INT>{1:2}; -- {1:2} を返します。
select map<INT,INT>{"12": "100"}; -- {12:100} を返します。
```

Key と Value は NULL にすることができます。

```SQL
select map{1:NULL};
```

空の Map を構築します。

```SQL
select map{} as empty_map;
select map() as empty_map; -- {} を返します。
```

## MAP 型データのインポート

StarRocks に MAP データをインポートするには、[INSERT INTO](../data-manipulation/INSERT.md) と [ORC/Parquet ファイルインポート](../data-manipulation/BROKER_LOAD.md) の二つの方法があります。

インポート中に StarRocks は重複する Key 値を削除します。

### INSERT INTO でのインポート

```SQL
  CREATE TABLE t0(
    c0 INT,
    c1 MAP<INT,INT>
  )
  DUPLICATE KEY(c0);
  
  INSERT INTO t0 VALUES(1, map{1:2,3:NULL});
```

### ORC と Parquet ファイルからのインポート

StarRocks の MAP 型は ORC と Parquet 形式のファイル内の MAP に対応しており、追加の変換や定義を行う必要はありません。具体的なインポート操作については、[Broker Load ドキュメント](../data-manipulation/BROKER_LOAD.md)を参照してください。

## MAP データのクエリ

**例 1：**テーブル `t0` の MAP 列 `c1` をクエリします。

```Plain
mysql> select c1 from t0;
+--------------+
| c1           |
+--------------+
| {1:2,3:null} |
+--------------+
```

**例 2：**`[ ]` 演算子または `element_at(any_map, any_key)` 関数を使用して、key に対応する value をクエリします。

以下の二つの例では Key `1` に対応する Value をクエリします。

```Plain
mysql> select map{1:2,3:NULL}[1];
+-----------------------+
| map(1, 2, 3, NULL)[1] |
+-----------------------+
|                     2 |
+-----------------------+

mysql> select element_at(map{1:2,3:NULL},1);
+--------------------+
| map{1:2,3:NULL}[1] |
+--------------------+
|                  2 |
+--------------------+
```

もし Key が存在しない場合は NULL を返します。

```Plain
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

**例 3：**複雑な MAP の要素を再帰的にクエリします。以下の例ではまず Key `1` に対応する値、つまり `map{2:1}` をクエリします。その後でその Map の中の Key `2` に対応する値をさらにクエリします。

```Plain Text
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 関連参照

- [Map 関数](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)

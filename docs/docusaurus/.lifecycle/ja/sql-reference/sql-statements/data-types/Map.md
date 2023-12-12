---
displayed_sidebar: "Japanese"
---

# MAP

## 説明

MAPは、キーと値の組のセットを格納する複雑なデータ型であり、例えば、`{a:1, b:2, c:3}`のようになります。 MAP内のキーはユニークでなければなりません。 ネストされたMAPは、最大14レベルのネストを含めることができます。

MAPデータ型はv3.1からサポートされています。 v3.1では、StarRocksテーブルを作成し、そのテーブルにMAPデータをロードし、MAPデータをクエリできます。

v2.5以降、StarRocksはデータレイクからの複雑なデータ型MAPおよびSTRUCTのクエリをサポートしています。 StarRocksが提供する外部カタログを使用して、Apache Hive™、Apache Hudi、およびApache IcebergからMAPおよびSTRUCTデータをクエリできます。 ORCおよびParquetファイルからのデータのみをクエリできます。 外部データソースをクエリする外部カタログの使用方法については、[カタログの概要](../../../data_source/catalog/catalog_overview.md)および必要なカタログタイプに関連するトピックを参照してください。

## 構文

```Haskell
MAP<key_type,value_type>
```

- `key_type`: キーのデータ型。 キーは、StarRocksでサポートされているプリミティブ型でなければなりません。 たとえば、数値、文字列、または日付です。 HLL、JSON、ARRAY、MAP、BITMAP、STRUCTタイプではあってはなりません。
- `value_type`: 値のデータ型。 値は任意のサポートされている型であることができます。

キーと値は**ネイティブにヌラブル**です。

## StarRocksでMAP列を定義する

この列にMAP型を持つテーブルを作成し、MAPデータをこの列にロードすることができます。

```SQL
-- 一次元のMAPを定義する。
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- ネストされたMAPを定義する。
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- NOT NULL MAPを定義する。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

MAPのデータ型を持つ列には、次の制限があります。

- テーブルのキー列として使用することはできません。 値としてのみ使用できます。
- テーブルのパーティションキー列（PARTITION BYの後に続く列）として使用することはできません。
- テーブルのバケット列（DISTRIBUTED BYの後に続く列）として使用することはできません。

## SQLでMAPを構築する

Mapは、次の2つの構文を使用してSQLで構築することができます。

- `map{key_expr:value_expr, ...}`: マップの要素はカンマ（`,`）で区切られ、キーと値はコロン（`:`）で区切られます。 例えば、`map{a:1, b:2, c:3}`です。

- `map(key_expr, value_expr ...)`: キーと値の式はペアでなければならず、 例えば、`map(a,1,b,2,c,3)`です。

StarRocksは、すべての入力のキーと値からキーと値のデータ型を導出することができます。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- Return {1:1,2:2,3:3}.
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- Return {1:"apple",2:"orange",3:"pear"}.
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- Return {1:{3.13:"abc"},0:{}}.
```

キーまたは値が異なる型の場合、StarRocksは適切な型（上位型）を自動的に導出します。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- Return {1.0:2.2,1.2:21.0}.
select map{12:"a", "100":1, NULL:NULL} as string_string; -- Return {"12":"a","100":"1",null:null}.
```

マップを構成する場合は、`<>`を使用してデータ型を定義することもできます。 入力のキーまたは値は指定された型にキャストできる必要があります。

```SQL
select map<FLOAT,INT>{1:2}; -- Return {1:2}.
select map<INT,INT>{"12": "100"}; -- Return {12:100}.
```

キーと値はヌラブルです。

```SQL
select map{1:NULL};
```

空のマップを構築します。

```SQL
select map{} as empty_map;
select map() as empty_map; -- Return {}.
```

## MAPデータをStarRocksにロードする

MAPデータをStarRocksにロードするには、[INSERT INTO](../../../loading/InsertInto.md)および[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の2つのメソッドを使用できます。

StarRocksは、MAPデータをロードする際に、各マップの重複するキーを削除します。

### INSERT INTO
```SQL
  CREATE TABLE t0(
    c0 INT,
    c1 MAP<INT,INT>
  )
  DUPLICATE KEY(c0);

  INSERT INTO t0 VALUES(1, map{1:2,3:NULL});
```

### ORCおよびParquetファイルからMAPデータをロードする

StarRocksのMAPデータ型は、ORCまたはParquet形式のマップ構造に対応しています。 追加の仕様は必要ありません。 [ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の手順に従って、ORCまたはParquetファイルからMAPデータをロードすることができます。

## MAPデータにアクセスする

例1: テーブル`t0`からMAP列`c1`をクエリします。

```Plain Text
mysql> select c1 from t0;
+--------------+
| c1           |
+--------------+
| {1:2,3:null} |
+--------------+
```

例2: `[ ]`演算子を使用してキーによってマップから値を取得するか、`element_at(any_map, any_key)`関数を使用する。

次の例では、キー`1`に対応する値をクエリします。

```Plain Text
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

キーがマップ内に存在しない場合、`NULL`が返されます。

次の例では、存在しないキーである2に対忋する値をクエリします。

```Plain Text
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

例3: **再帰的に**多次元マップをクエリします。

次の例では、まずキー`1`に対応する値である`map{2:1}`をクエリし、その後、`map{2:1}`内のキー`2`に再帰的にクエリします。

```Plain Text
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 参照

- [Map functions](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)
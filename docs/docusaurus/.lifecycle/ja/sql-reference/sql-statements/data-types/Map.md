---
displayed_sidebar: "Japanese"
---

# MAP

## 説明

MAPは、キーと値の組を複数格納する複雑なデータ型です。たとえば、`{a:1, b:2, c:3}`のようになります。MAP内のキーはユニークである必要があります。ネストされたMAPは最大14レベルのネストを含むことができます。

MAPデータ型はv3.1以降でサポートされています。v3.1では、MAP列をStarRocksテーブルを作成し、そのテーブルにMAPデータをロードし、MAPデータをクエリできます。

v2.5以降、StarRocksはデータレイクからの複雑なデータ型MAPとSTRUCTのクエリをサポートしています。Apache Hive™、Apache Hudi、Apache Icebergから提供される外部カタログを使用して、MAPおよびSTRUCTデータをクエリできます。ORCおよびParquetファイルからのデータのみをクエリできます。外部カタログを使用して外部データソースをクエリする方法の詳細については、[カタログの概要](../../../data_source/catalog/catalog_overview.md)および必要なカタログタイプに関連するトピックをご覧ください。

## 構文

```Haskell
MAP<key_type,value_type>
```

- `key_type`: キーのデータ型。キーは、StarRocksでサポートされているプリミティブな型（数値、文字列、日付など）でなければなりません。HLL、JSON、ARRAY、MAP、BITMAP、STRUCT型はキーにできません。
- `value_type`: 値のデータ型。値はどんなサポートされている型でも構いません。

キーと値は**ネイティブにnullable**です。

## StarRocksでMAP列を定義する

テーブルを作成し、この列にMAPデータをロードする際に、MAP列を定義することができます。

```SQL
-- 1次元のMAPを定義。
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- ネストされたMAPを定義。
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- NOT NULLなMAPを定義。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

MAP型の列には以下の制限があります：

- テーブルのキーカラムとして使用できません。値列としてのみ使用できます。
- テーブルのパーティションキーカラム (PARTITION BYに続く列) として使用できません。
- テーブルのバケティングカラム (DISTRIBUTED BYに続く列) として使用できません。

## SQLでMAPを構築する

SQLでMAPは以下の2つの構文を使用して構築できます：

- `map{key_expr:value_expr, ...}`: MAP要素はカンマ(`,`)で区切られ、キーと値はコロン(`:`)で区切られます。たとえば、`map{a:1, b:2, c:3}`。

- `map(key_expr, value_expr ...)`: キーと値の式はペアでなければなりません。たとえば、`map(a,1,b,2,c,3)`。

StarRocksは、すべての入力キーと値からキーと値のデータ型を導出します。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- {1:1,2:2,3:3} を返します。
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- {1:"apple",2:"orange",3:"pear"} を返します。
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- {1:{3.13:"abc"},0:{}} を返します。
```

キーまたは値が異なる型の場合、StarRocksは適切な型（サブタイプ）を自動的に導出します。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- {1.0:2.2,1.2:21.0} を返します。
select map{12:"a", "100":1, NULL:NULL} as string_string; -- {"12":"a","100":"1",null:null} を返します。
```

また、マップを構築する際に`<>`を使用してデータ型を定義することもできます。入力キーまたは値は指定された型にキャストできる必要があります。

```SQL
select map<FLOAT,INT>{1:2}; -- {1:2} を返します。
select map<INT,INT>{"12": "100"}; -- {12:100} を返します。
```

キーと値はnullableです。

```SQL
select map{1:NULL};
```

空のMAPを構築することもできます。

```SQL
select map{} as empty_map;
select map() as empty_map; -- {} を返します。
```

## MAPデータをStarRocksにロードする

MAPデータをStarRocksに読み込む方法は、[INSERT INTO](../../../loading/InsertInto.md)と[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の2つがあります。

StarRocksは、マップ毎の重複するキーを削除します。

### INSERT INTO

```SQL
  CREATE TABLE t0(
    c0 INT,
    c1 MAP<INT,INT>
  )
  DUPLICATE KEY(c0);

  INSERT INTO t0 VALUES(1, map{1:2,3:NULL});
```

### ORCおよびParquetファイルからMAPデータを読み込む

StarRocksのMAPデータ型はORCやParquetの構造に対応しています。追加の仕様は必要ありません。[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の手順に従って、ORCやParquetファイルからMAPデータを読み込むことができます。

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

例2: `[ ]`演算子または`element_at(any_map, any_key)`関数を使用して、マップからキーに対応する値を取得します。

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

マップ内にキーが存在しない場合、`NULL`が返されます。

次の例は、存在しないキー2に対応する値をクエリします。

```Plain Text
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

例3: マルチディメンショナルマップを**再帰的に**クエリします。

次の例では、まずキー`1`に対応する値である`map{2:1}`をクエリし、その後`map{2:1}`内のキー`2`に再帰的にクエリします。

```Plain Text
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 参照

- [Map関数](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)
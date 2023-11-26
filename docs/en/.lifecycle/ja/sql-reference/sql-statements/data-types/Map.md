---
displayed_sidebar: "Japanese"
---

# MAP

## 説明

MAPは、キーと値のペアのセットを格納する複雑なデータ型です。例えば、`{a:1, b:2, c:3}`のような形式です。MAP内のキーは一意である必要があります。ネストされたMAPは最大14階層のネストを持つことができます。

MAPデータ型はv3.1以降でサポートされています。v3.1では、StarRocksテーブルを作成する際にMAP列を定義し、そのテーブルにMAPデータをロードしてMAPデータをクエリすることができます。

v2.5以降、StarRocksはデータレイクからのMAPおよびSTRUCTのクエリをサポートしています。StarRocksが提供する外部カタログを使用して、Apache Hive™、Apache Hudi、およびApache IcebergからMAPおよびSTRUCTデータをクエリすることができます。ORCおよびParquetファイルからのデータのみクエリできます。外部データソースをクエリするための外部カタログの使用方法の詳細については、[カタログの概要](../../../data_source/catalog/catalog_overview.md)および必要なカタログタイプに関連するトピックを参照してください。

## 構文

```Haskell
MAP<key_type,value_type>
```

- `key_type`: キーのデータ型。キーは、StarRocksでサポートされているプリミティブなデータ型（数値、文字列、日付など）である必要があります。HLL、JSON、ARRAY、MAP、BITMAP、STRUCTのいずれかの型ではないことが必要です。
- `value_type`: 値のデータ型。値は、任意のサポートされている型であることができます。

キーと値は**ネイティブにnull可能**です。

## StarRocksでMAP列を定義する

テーブルを作成し、この列にMAPデータをロードする際に、MAP列を定義することができます。

```SQL
-- 1次元のMAPを定義する。
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

-- NOT NULLなMAPを定義する。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

MAP型の列には以下の制限があります。

- テーブルのキーカラムとして使用することはできません。値カラムとしてのみ使用できます。
- テーブルのパーティションキーカラム（PARTITION BYの後に続く列）として使用することはできません。
- テーブルのバケット化カラム（DISTRIBUTED BYの後に続く列）として使用することはできません。

## SQLでMAPを構築する

MAPは、次の2つの構文を使用してSQLで構築することができます。

- `map{key_expr:value_expr, ...}`: MAPの要素はコンマ（`,`）で区切られ、キーと値はコロン（`:`）で区切られます。例えば、`map{a:1, b:2, c:3}`です。

- `map(key_expr, value_expr ...)`: キーと値の式はペアである必要があります。例えば、`map(a,1,b,2,c,3)`です。

StarRocksは、すべての入力キーと値からキーと値のデータ型を推定することができます。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- {1:1,2:2,3:3}を返します。
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- {1:"apple",2:"orange",3:"pear"}を返します。
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- {1:{3.13:"abc"},0:{}}を返します。
```

キーまたは値の型が異なる場合、StarRocksは適切な型（スーパータイプ）を自動的に推定します。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- {1.0:2.2,1.2:21.0}を返します。
select map{12:"a", "100":1, NULL:NULL} as string_string; -- {"12":"a","100":"1",null:null}を返します。
```

また、マップを構築する際に`<>`を使用してデータ型を定義することもできます。入力のキーまたは値は指定された型にキャストできる必要があります。

```SQL
select map<FLOAT,INT>{1:2}; -- {1:2}を返します。
select map<INT,INT>{"12": "100"}; -- {12:100}を返します。
```

キーと値はnull可能です。

```SQL
select map{1:NULL};
```

空のマップを構築します。

```SQL
select map{} as empty_map;
select map() as empty_map; -- {}を返します。
```

## MAPデータをStarRocksにロードする

MAPデータをStarRocksには、[INSERT INTO](../../../loading/InsertInto.md)および[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の2つの方法を使用してロードすることができます。

MAPデータをロードする際に、StarRocksは各マップの重複するキーを削除します。

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

StarRocksのMAPデータ型は、ORCまたはParquet形式のマップ構造に対応しています。追加の指定は必要ありません。ORCまたはParquetファイルからMAPデータをロードするには、[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の手順に従ってください。

## MAPデータへのアクセス

例1: テーブル`t0`からMAP列`c1`をクエリします。

```Plain Text
mysql> select c1 from t0;
+--------------+
| c1           |
+--------------+
| {1:2,3:null} |
+--------------+
```

例2: `[ ]`演算子を使用してキーによってマップから値を取得するか、`element_at(any_map, any_key)`関数を使用します。

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

次の例では、存在しないキー2に対応する値をクエリします。

```Plain Text
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

例3: マルチディメンションのマップを**再帰的に**クエリします。

次の例では、まずキー`1`に対応する値である`map{2:1}`をクエリし、その後、`map{2:1}`内のキー`2`に対応する値を再帰的にクエリします。

```Plain Text
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 参考

- [Map functions](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)

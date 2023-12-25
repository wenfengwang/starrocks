---
displayed_sidebar: English
---

# MAP

## 説明

MAPは、キーと値のペアのセットを格納する複合データ型です。例えば、`{a:1, b:2, c:3}`のようになります。マップ内のキーは一意でなければなりません。ネストされたマップは最大14レベルのネストを含むことができます。

MAPデータ型はv3.1以降でサポートされています。v3.1では、StarRocksテーブルを作成する際にMAP列を定義し、そのテーブルにMAPデータをロードし、MAPデータをクエリすることができます。

v2.5以降、StarRocksはデータレイクからの複雑なデータ型MAPとSTRUCTのクエリをサポートしています。StarRocksが提供する外部カタログを使用して、Apache Hive™、Apache Hudi、Apache IcebergからMAPおよびSTRUCTデータをクエリできます。クエリはORCファイルとParquetファイルからのデータに限定されます。外部カタログを使用して外部データソースをクエリする方法の詳細については、[カタログの概要](../../../data_source/catalog/catalog_overview.md)および必要なカタログタイプに関連するトピックを参照してください。

## 構文

```Haskell
MAP<key_type,value_type>
```

- `key_type`: キーのデータ型。キーはStarRocksがサポートするプリミティブ型でなければなりません。例えば、数値、文字列、日付などです。HLL、JSON、ARRAY、MAP、BITMAP、STRUCTタイプは使用できません。
- `value_type`: 値のデータ型。値はサポートされている任意の型であることができます。

キーと値は**ネイティブにnull許容**です。

## StarRocksでのMAP列の定義

テーブルを作成する際にMAP列を定義し、この列にMAPデータをロードすることができます。

```SQL
-- 一次元マップを定義する。
CREATE TABLE t0(
  c0 INT,
  c1 MAP<INT,INT>
)
DUPLICATE KEY(c0);

-- ネストされたマップを定義する。
CREATE TABLE t1(
  c0 INT,
  c1 MAP<DATE, MAP<VARCHAR(10), INT>>
)
DUPLICATE KEY(c0);

-- NULLでないマップを定義する。
CREATE TABLE t2(
  c0 INT,
  c1 MAP<INT,DATETIME> NOT NULL
)
DUPLICATE KEY(c0);
```

MAP型の列には以下の制約があります：

- テーブルのキーカラムとして使用することはできません。値カラムとしてのみ使用できます。
- テーブルのパーティションキーカラム（PARTITION BYに続くカラム）としては使用できません。
- テーブルのバケッティングカラム（DISTRIBUTED BYに続くカラム）としては使用できません。

## SQLでのマップの構築

マップは、以下の2つの構文を使用してSQLで構築できます：

- `map{key_expr:value_expr, ...}`: マップの要素はコンマ(`,`)で区切られ、キーと値はコロン(`:`)で区切られます。例：`map{a:1, b:2, c:3}`。

- `map(key_expr, value_expr ...)`: キーと値の式はペアでなければなりません。例：`map(a,1,b,2,c,3)`。

StarRocksは、すべての入力キーと値からキーと値のデータ型を導出することができます。

```SQL
select map{1:1, 2:2, 3:3} as numbers;
select map(1,1,2,2,3,3) as numbers; -- {1:1,2:2,3:3}を返します。
select map{1:"apple", 2:"orange", 3:"pear"} as fruit;
select map(1, "apple", 2, "orange", 3, "pear") as fruit; -- {1:"apple",2:"orange",3:"pear"}を返します。
select map{true:map{3.13:"abc"}, false:map{}} as nest;
select map(true, map(3.13, "abc"), false, map{}) as nest; -- {true:{3.13:"abc"},false:{}}を返します。
```

キーまたは値が異なる型の場合、StarRocksは自動的に適切な型（スーパータイプ）を導出します。

```SQL
select map{1:2.2, 1.2:21} as floats_floats; -- {1.0:2.2,1.2:21.0}を返します。
select map{12:"a", "100":1, NULL:NULL} as string_string; -- {"12":"a","100":"1",null:null}を返します。
```

また、マップを構築する際に`<>`を使用してデータ型を定義することもできます。入力キーまたは値は指定された型にキャスト可能である必要があります。

```SQL
select map<FLOAT,INT>{1:2}; -- {1:2}を返します。
select map<INT,INT>{"12": "100"}; -- {12:100}を返します。
```

キーと値はnull許容です。

```SQL
select map{1:NULL};
```

空のマップを構築します。

```SQL
select map{} as empty_map;
select map() as empty_map; -- {}を返します。
```

## StarRocksへのMAPデータのロード

StarRocksにMAPデータをロードするには、[INSERT INTO](../../../loading/InsertInto.md)と[ORC/Parquetロード](../data-manipulation/BROKER_LOAD.md)の2つの方法があります。

StarRocksはMAPデータをロードする際に、各マップの重複するキーを削除します。

### INSERT INTO

```SQL
  CREATE TABLE t0(
    c0 INT,
    c1 MAP<INT,INT>
  )
  DUPLICATE KEY(c0);

  INSERT INTO t0 VALUES(1, map{1:2,3:NULL});
```

### ORCファイルとParquetファイルからのMAPデータのロード

StarRocksのMAPデータ型はORCまたはParquet形式のマップ構造に対応しています。追加の指定は必要ありません。[ORC/Parquetロード](../data-manipulation/BROKER_LOAD.md)の手順に従ってORCまたはParquetファイルからMAPデータをロードできます。

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

例2: `[]`演算子を使用してキーによるマップからの値の取得、または`element_at(any_map, any_key)`関数の使用。

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

キーがマップに存在しない場合、`NULL`が返されます。

次の例では、存在しないキー`2`に対応する値をクエリします。

```Plain Text
mysql> select map{1:2,3:NULL}[2];
+-----------------------+
| map(1, 2, 3, NULL)[2] |
+-----------------------+
|                  NULL |
+-----------------------+
```

例3: 多次元マップを**再帰的にクエリ**します。

次の例では、最初にキー`1`に対応する値`map{2:1}`をクエリし、次に`map{2:1}`内のキー`2`に対応する値を再帰的にクエリします。

```Plain Text
mysql> select map{1:map{2:1},3:NULL}[1][2];

+----------------------------------+
| map(1, map(2, 1), 3, NULL)[1][2] |
+----------------------------------+
|                                1 |
+----------------------------------+
```

## 参照

- [マップ関数](../../sql-functions/map-functions/map_values.md)
- [element_at](../../sql-functions/array-functions/element_at.md)

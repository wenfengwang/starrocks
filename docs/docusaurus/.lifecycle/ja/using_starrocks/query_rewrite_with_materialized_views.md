---
displayed_sidebar: "Japanese"
---

# マテリアライズドビューを使用したクエリの書き換え

このトピックでは、StarRocksの非同期マテリアライズドビューを活用してクエリを書き換え、加速させる方法について説明します。

## 概要

StarRocksの非同期マテリアライズドビューは、SPJG（select-project-join-group-by）形式に基づく透過的なクエリ書き換えアルゴリズムを使用しています。クエリ文を変更する必要なしに、StarRocksはベーステーブルに対するクエリを自動的に、事前に計算された結果を含むマテリアライズドビューに対するクエリに書き換えることができます。その結果、マテリアライズドビューは計算コストを大幅に削減し、クエリの実行を大幅に加速するのに役立ちます。

非同期マテリアライズドビューに基づくクエリ書き換え機能は、特に以下のシナリオで有用です:

- **メトリクスの事前集約**

  データの次元が高い場合、マテリアライズドビューを使用して事前に集約されたメトリクス層を作成できます。

- **ワイドテーブルの結合**

  マテリアライズドビューを使用すると、複雑なシナリオで複数の大規模なワイドテーブルを結合したクエリを透過的に高速化することができます。

- **データレイクでのクエリの加速**

  外部カタログベースのマテリアライズドビューを構築することで、データレイクのデータに対するクエリを簡単に高速化することができます。

  > **注意**
  >
  > JDBCカタログ上のベーステーブルに作成された非同期マテリアライズドビューは、クエリの書き換えをサポートしません。

### 特徴

StarRocksの非同期マテリアライズドビューに基づく自動クエリ書き換え機能は、以下の属性を持っています:

- **強力なデータの整合性**: ベーステーブルがネイティブテーブルの場合、StarRocksはマテリアライズドビューに基づくクエリ書き換えを通じて得られた結果が、ベーステーブルに対する直接のクエリから返される結果と整合性があることを保証します。
- **古さの書き直し**: StarRocksは古さの書き直しをサポートし、頻繁なデータ変更のシナリオに対処するために、ある程度のデータの有効期限を許容することができます。
- **複数テーブルの結合**: StarRocksの非同期マテリアライズドビューは、View Delta JoinsやDerivable Joinsなどの複雑な結合シナリオを含むさまざまな結合タイプをサポートし、大規模なワイドテーブルを含むシナリオでのクエリの加速を可能にします。
- **集約の書き直し**: StarRocksは集約を伴うクエリを書き直して、レポートのパフォーマンスを向上させることができます。
- **ネストしたマテリアライズドビュー**: StarRocksは、ネストしたマテリアライズドビューに基づく複雑なクエリを書き直すことができ、書き直せるクエリの範囲を拡大することができます。
- **Unionの書き直し**: Unionの書き直し機能とマテリアライズドビューのパーティションのTTL（有効期限）を組み合わせることで、ホットデータとコールドデータの分離を実現し、マテリアライズドビューからホットデータをクエリし、ベーステーブルから過去のデータを取得することができます。
- **ビュー上のマテリアライズドビュー**: ビューに基づくデータモデリングシナリオでのクエリを高速化することができます。
- **外部カタログ上のマテリアライズドビュー**: データレイクでのクエリを高速化することができます。
- **複雑な式の書き直し**: 関数呼び出しや演算など、複雑な式を処理することができ、高度な分析や計算要件に対応することができます。

これらの特徴は、次のセクションで詳しく説明されます。

## 結合の書き直し

StarRocksは、内部結合、クロス結合、左外部結合、右外部結合、セミ結合、アンチ結合など、さまざまなタイプの結合を含むクエリを書き直すことをサポートしています。

以下は、結合を含むクエリを書き直す例です。次のように2つのベーステーブルを作成します:

```SQL
CREATE TABLE customer (
  c_custkey     INT(11)     NOT NULL,
  c_name        VARCHAR(26) NOT NULL,
  c_address     VARCHAR(41) NOT NULL,
  c_city        VARCHAR(11) NOT NULL,
  c_nation      VARCHAR(16) NOT NULL,
  c_region      VARCHAR(13) NOT NULL,
  c_phone       VARCHAR(16) NOT NULL,
  c_mktsegment  VARCHAR(11) NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(c_custkey)
DISTRIBUTED BY HASH(c_custkey) BUCKETS 12;

CREATE TABLE lineorder (
  lo_orderkey         INT(11) NOT NULL,
  lo_linenumber       INT(11) NOT NULL,
  lo_custkey          INT(11) NOT NULL,
  lo_partkey          INT(11) NOT NULL,
  lo_suppkey          INT(11) NOT NULL,
  lo_orderdate        INT(11) NOT NULL,
  lo_orderpriority    VARCHAR(16) NOT NULL,
  lo_shippriority     INT(11) NOT NULL,
  lo_quantity         INT(11) NOT NULL,
  lo_extendedprice    INT(11) NOT NULL,
  lo_ordtotalprice    INT(11) NOT NULL,
  lo_discount         INT(11) NOT NULL,
  lo_revenue          INT(11) NOT NULL,
  lo_supplycost       INT(11) NOT NULL,
  lo_tax              INT(11) NOT NULL,
  lo_commitdate       INT(11) NOT NULL,
  lo_shipmode         VARCHAR(11) NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(lo_orderkey)
DISTRIBUTED BY HASH(lo_orderkey) BUCKETS 48;
```

上記のベーステーブルを使用して、次のようにマテリアライズドビューを作成できます:

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY HASH(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, lo_partkey, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

このようなマテリアライズドビューは、次のクエリを書き換えることができます:

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

![Rewrite-1](../assets/Rewrite-1.png)

StarRocksは、演算、文字列関数、日付関数、CASE WHEN式、OR述語など、複雑な式を含む結合クエリを書き直すことがサポートしています。例えば、上記のマテリアライズドビューは、次のクエリを書き換えることができます:

```SQL
SELECT 
    lo_orderkey, 
    lo_linenumber, 
    (2 * lo_revenue + 1) * lo_linenumber, 
    upper(c_name), 
    substr(c_address, 3)
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

従来のシナリオに加えて、StarRocksはさらに複雑なシナリオでの結合クエリの書き直しをサポートしています。

### クエリデルタ結合の書き直し

クエリデルタ結合とは、クエリで結合されるテーブルがマテリアライズドビューで結合されるテーブルの上位集合であるシナリオを指します。例えば、次のような3つのテーブル `lineorder`, `customer`, `part` を結合するクエリを考えます。マテリアライズドビュー `join_mv1` には `lineorder` と `customer` の結合のみが含まれる場合、StarRocksはこのクエリを `join_mv1` を使用して書き換えることができます。

例:

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address, p_name
FROM
    lineorder INNER JOIN customer ON lo_custkey = c_custkey
    INNER JOIN part ON lo_partkey = p_partkey;
```

その元のクエリプランと書き換え後のクエリプランは次のとおりです:

![Rewrite-2](../assets/Rewrite-2.png)

### ビューデルタ結合の書き直し

ビューデルタ結合とは、クエリで結合されるテーブルがマテリアライズドビューで結合されるテーブルのサブセットであるシナリオを指します。この機能は、通常は大規模なワイドテーブルを含むシナリオで使用されます。例えば、Star Schema Benchmark（SSB）のコンテキストでは、すべてのテーブルを結合するマテリアライズドビューを作成してクエリのパフォーマンスを向上させることができます。テストの結果、複数テーブルを結合するクエリのパフォーマンスは、マテリアライズドビューを介してクエリを透過的に書き換えた後も、対応する大規模なワイドテーブルをクエリするレベルと同程度になることがわかりました。

ビューデルタ結合を行うためには、マテリアライズドビューには、クエリに存在しない1:1の標準保存結合が含まれている必要があります。以下に、標準保存結合と見なされる9種類の結合と、それらのいずれかを満たすことでビューデルタ結合の書き直しが可能となります:

![Rewrite-3](../assets/Rewrite-3.png)

SSBテストを例にとって、次のようにベーステーブルを作成します:

```SQL
CREATE TABLE customer (
  c_custkey         INT(11)       NOT NULL,
  c_name            VARCHAR(26)   NOT NULL,
  c_address         VARCHAR(41)   NOT NULL,
  c_city            VARCHAR(11)   NOT NULL,
  c_nation          VARCHAR(16)   NOT NULL,
  c_region          VARCHAR(13)   NOT NULL,
  c_phone           VARCHAR(16)   NOT NULL,
  c_mktsegment      VARCHAR(11)   NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(c_custkey)
DISTRIBUTED BY HASH(c_custkey) BUCKETS 12
PROPERTIES (
"unique_constraints" = "c_custkey"   -- 一意の制約を指定します。
);

CREATE TABLE dates (
  d_datekey          DATE          NOT NULL,
  d_date             VARCHAR(20)   NOT NULL,
  d_dayofweek        VARCHAR(10)   NOT NULL,
  d_month            VARCHAR(11)   NOT NULL,
  d_year             INT(11)       NOT NULL,
  d_yearmonthnum     INT(11)       NOT NULL,
  d_yearmonth        VARCHAR(9)    NOT NULL,
  d_daynuminweek     INT(11)       NOT NULL,
  d_daynuminmonth    INT(11)       NOT NULL,
  d_daynuminyear     INT(11)       NOT NULL,
  d_monthnuminyear   INT(11)       NOT NULL,
```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH MANUAL
PROPERTIES (
    "partition_refresh_number"="1"
)
AS SELECT /*+ SET_VAR(query_timeout = 7200) */     -- Set timeout for the refresh operation.
       l.LO_ORDERDATE        AS LO_ORDERDATE,
       l.LO_ORDERKEY         AS LO_ORDERKEY,
       l.LO_LINENUMBER       AS LO_LINENUMBER,
       l.LO_CUSTKEY          AS LO_CUSTKEY,
       l.LO_PARTKEY          AS LO_PARTKEY,
       l.LO_SUPPKEY          AS LO_SUPPKEY,
       l.LO_ORDERPRIORITY    AS LO_ORDERPRIORITY,
       l.LO_SHIPPRIORITY     AS LO_SHIPPRIORITY,
       l.LO_QUANTITY         AS LO_QUANTITY,
       l.LO_EXTENDEDPRICE    AS LO_EXTENDEDPRICE,
       l.LO_ORDTOTALPRICE    AS LO_ORDTOTALPRICE,
       l.LO_DISCOUNT         AS LO_DISCOUNT,
       l.LO_REVENUE          AS LO_REVENUE,
       l.LO_SUPPLYCOST       AS LO_SUPPLYCOST,
       l.LO_TAX              AS LO_TAX,
       l.LO_COMMITDATE       AS LO_COMMITDATE,
       l.LO_SHIPMODE         AS LO_SHIPMODE,
       c.C_NAME              AS C_NAME,
       c.C_ADDRESS           AS C_ADDRESS,
       c.C_CITY              AS C_CITY,
       c.C_NATION            AS C_NATION,
       c.C_REGION            AS C_REGION,
       c.C_PHONE             AS C_PHONE,
       c.C_MKTSEGMENT        AS C_MKTSEGMENT,
       s.S_NAME              AS S_NAME,
       s.S_ADDRESS           AS S_ADDRESS,
       s.S_CITY              AS S_CITY,
       s.S_NATION            AS S_NATION,
       s.S_REGION            AS S_REGION,
       s.S_PHONE             AS S_PHONE,
       p.P_NAME              AS P_NAME,
       p.P_MFGR              AS P_MFGR,
       p.P_CATEGORY          AS P_CATEGORY,
       p.P_BRAND             AS P_BRAND,
       p.P_COLOR             AS P_COLOR,
       p.P_TYPE              AS P_TYPE,
       p.P_SIZE              AS P_SIZE,
       p.P_CONTAINER         AS P_CONTAINER,
       d.D_DATE              AS D_DATE,
       d.D_DAYOFWEEK         AS D_DAYOFWEEK,
       d.D_MONTH             AS D_MONTH,
       d.D_YEAR              AS D_YEAR,
       d.D_YEARMONTHNUM      AS D_YEARMONTHNUM,
       d.D_YEARMONTH         AS D_YEARMONTH,
       d.D_DAYNUMINWEEK      AS D_DAYNUMINWEEK,
       d.D_DAYNUMINMONTH     AS D_DAYNUMINMONTH,
       d.D_DAYNUMINYEAR      AS D_DAYNUMINYEAR,
       d.D_MONTHNUMINYEAR    AS D_MONTHNUMINYEAR,
       d.D_WEEKNUMINYEAR     AS D_WEEKNUMINYEAR,
       d.D_SELLINGSEASON     AS D_SELLINGSEASON,
       d.D_LASTDAYINWEEKFL   AS D_LASTDAYINWEEKFL,
       d.D_LASTDAYINMONTHFL  AS D_LASTDAYINMONTHFL,
       d.D_HOLIDAYFL         AS D_HOLIDAYFL,
       d.D_WEEKDAYFL         AS D_WEEKDAYFL
   FROM lineorder            AS l
       INNER JOIN customer   AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
       INNER JOIN supplier   AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
       INNER JOIN part       AS p ON p.P_PARTKEY = l.LO_PARTKEY
       INNER JOIN dates      AS d ON l.LO_ORDERDATE = d.D_DATEKEY;    
```
同様に、SSBの他のクエリも`lineorder_flat_mv`を使用して透過的に書き換えることができ、クエリのパフォーマンスを最適化できます。

### 結合可能リライト

結合可能性リライトとは、マテリアライズド ビューとクエリの結合タイプが一致しないが、マテリアライズド ビューの結合結果にはクエリの結合結果が含まれているシナリオを指します。現在は、3つ以上のテーブルを結合するシナリオと、2つのテーブルを結合するシナリオの2つのシナリオがサポートされています。

- **シナリオ1：3つ以上のテーブルを結合する**

  たとえば、マテリアライズド ビューにはテーブル `t1` と `t2` の Left Outer Join およびテーブル `t2` と `t3` の Inner Join が含まれているとします。両方の結合で、結合条件には `t2` の列が含まれています。

  一方、クエリには、t1 と t2 の Inner Join および t2 と t3 の Inner Join が含まれています。両方の結合で、結合条件には t2 の列が含まれています。

  この場合、クエリはマテリアライズド ビューを使用して書き換えることができます。なぜなら、マテリアライズド ビューでは、Left Outer Join が最初に実行され、後に Inner Join が続きます。Left Outer Join によって生成された右側のテーブルには結果がない（つまり、右側のテーブルの列は NULL です）。これらの結果は次に Inner Join でフィルタリングされます。したがって、マテリアライズド ビューとクエリの論理は等価であり、クエリを書き換えることができます。

  例：

  マテリアライズド ビュー `join_mv5` を作成します：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv5
  PARTITION BY lo_orderdate
  DISTRIBUTED BY hash(lo_orderkey)
  PROPERTIES (
    "partition_refresh_number" = "1"
  )
  AS
  SELECT lo_orderkey, lo_orderdate, lo_linenumber, lo_revenue, c_custkey, c_address, p_name
  FROM customer LEFT OUTER JOIN lineorder
  ON c_custkey = lo_custkey
  INNER JOIN part
  ON p_partkey = lo_partkey;
  ```

  `join_mv5` は次のクエリを書き換えることができます：

  ```SQL
  SELECT lo_orderkey, lo_orderdate, lo_linenumber, lo_revenue, c_custkey, c_address, p_name
  FROM customer INNER JOIN lineorder
  ON c_custkey = lo_custkey
  INNER JOIN part
  ON p_partkey = lo_partkey;
  ```

  書き換え前のクエリプランと書き換え後のクエリプランは次のようになります：

  ![Rewrite-5](../assets/Rewrite-5.png)

  同様に、マテリアライズド ビューが `t1 INNER JOIN t2 INNER JOIN t3` と定義され、クエリが `LEFT OUTER JOIN t2 INNER JOIN t3` である場合、クエリも書き換えることができます。さらに、この書き換え機能は3つ以上のテーブルを含むシナリオにも拡張されます。

- **シナリオ2：2つのテーブルを結合する**

  2つのテーブルを関連付ける結合可能性リライト機能は、特定のケースをサポートしています：

  ![Rewrite-6](../assets/Rewrite-6.png)

  ケース1から9では、意味的等価性を確保するために、書き換えられた結果にフィルタリング述語を追加する必要があります。たとえば、次のようにマテリアライズド ビューを作成します：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv3
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  次のクエリは`join_mv3`を使用して書き換えることができ、書き換え結果には `c_custkey IS NOT NULL` というフィルタリング述語が追加されます：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  書き換え前のクエリプランと書き換え後のクエリプランは次のようになります：

  ![Rewrite-7](../assets/Rewrite-7.png)

  ケース10では、Left Outer Join クエリに右側のテーブルにフィルタリング述語 `IS NOT NULL` を含める必要があります。たとえば、次のようにマテリアライズド ビューを作成します：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv4
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  `join_mv4`は次のクエリを書き換えることができ、`customer.c_address = "Sb4gxKs7"` がフィルタリング述語 `IS NOT NULL` です：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey
  WHERE customer.c_address = "Sb4gxKs7";
  ```

  書き換え前のクエリプランと書き換え後のクエリプランは次のようになります：

  ![Rewrite-8](../assets/Rewrite-8.png)

## 集約リライト

StarRocks の非同期マテリアライズド ビューは、bitmap_union、hll_union、およびpercentile_unionを含むすべての利用可能な集約関数を持つ複数テーブルの集計クエリを書き換えることができます。たとえば、次のようにマテリアライズド ビューを作成します：

```SQL
CREATE MATERIALIZED VIEW agg_mv1
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  lo_linenumber, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey, lo_linenumber, c_name;
```

これは次のクエリを書き換えることができます：

```SQL
SELECT 
  lo_orderkey, 
  lo_linenumber, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey, lo_linenumber, c_name;
```

書き換え前のクエリプランと書き換え後のクエリプランは次のようになります：

![Rewrite-9](../assets/Rewrite-9.png)

次のセクションでは、集約リライト機能が有用なシナリオについて詳しく説明します。

### 集約ロールアップリライト

StarRocks は、集計ロールアップを含むクエリを書き換える場合をサポートしています。つまり、StarRocks は、`GROUP BY a` 句を使用して作成された非同期マテリアライズド ビューを使用して `GROUP BY a,b` 句を使用して作成された集計クエリを書き換えることができます。たとえば、次のクエリは`agg_mv1`を使用して書き換えることができます：

```SQL
SELECT 
  lo_orderkey, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey, c_name;
```

書き換え前のクエリプランと書き換え後のクエリプランは次のようになります：

![Rewrite-10](../assets/Rewrite-10.png)

> **注意**
>
> 現在、グループ化セット、GROUPING SET with ROLLUP、または GROUPING SET with CUBE を書き換えることはサポートされていません。

一部の集約関数のみが集約ロールアップでクエリを書き換えることをサポートしています。前述の例では、`count(distinct client_id)` の代わりに `bitmap_union(to_bitmap(client_id))` を使用するマテリアライズド ビュー `order_agg_mv` がある場合、StarRocks は集計ロールアップを持つクエリを書き換えることができません。

次の表は、オリジナルのクエリの集約関数と、マテリアライズド ビューで使用される集約関数の対応関係を示しています。ビジネスシナリオに応じて、適切な集約関数を選択してマテリアライズド ビューを構築することができます。

| **オリジナル クエリでサポートされる集約関数** | **マテリアライズド ビューでサポートされる集約関数** |
| ---------------------------------------- | -------------------------------------------- |
| sum                                      | sum                                          |
| count                                    | count                                        |
| min                                      | min                                          |
| max                                      | max                                          |
| avg                                      | sum / count                                  |
| bitmap_union, bitmap_union_count, count(distinct) | bitmap_union                       |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                          |
| percentile_approx, percentile_union      | percentile_union                             |

対応する GROUP BY 列がない場合、DISTINCT 集約は集約ロールアップを持つクエリを書き換えることはできません。ただし、StarRocks v3.1以降では、集約ロールアップDISTINCT集約関数を持つクエリがGROUP BY列を持たずに等しい述語を持つ場合、関連するマテリアライズド ビューによっても書き換えることが可能です。以下の例では、StarRocks は`order_agg_mv1`を使用してクエリを書き換えることができます。
-- Query
SELECT
    order_date,
    count(distinct client_id) 
FROM order_list WHERE order_date='2023-07-03';

### COUNT DISTINCT rewrite

StarRocksは、COUNT DISTINCTの計算をビットマップベースの計算に書き換えることをサポートしており、マテリアライズドビューを使用して高性能かつ正確な重複排除を可能にします。たとえば、次のようにマテリアライズドビューを作成します。

```SQL
CREATE MATERIALIZED VIEW distinct_mv
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, bitmap_union(to_bitmap(lo_custkey)) AS distinct_customer
FROM lineorder
GROUP BY lo_orderkey;
```

これにより次のクエリが書き換えられます。

```SQL
SELECT lo_orderkey, count(distinct lo_custkey) 
FROM lineorder 
GROUP BY lo_orderkey;
```

## ネストされたマテリアライズドビューの書き換え

StarRocksは、ネストされたマテリアライズドビューを使用するクエリを書き換えることをサポートしています。たとえば、次のようにマテリアライズドビュー`join_mv2`、`agg_mv2`、`agg_mv3`を作成します。

```SQL
CREATE MATERIALIZED VIEW join_mv2
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;


CREATE MATERIALIZED VIEW agg_mv2
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  lo_linenumber, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM join_mv2
GROUP BY lo_orderkey, lo_linenumber, c_name;

CREATE MATERIALIZED VIEW agg_mv3
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  sum(total_revenue) AS total_revenue, 
  max(max_discount) AS max_discount 
FROM agg_mv2
GROUP BY lo_orderkey;
```

これにより、`agg_mv3`が次のクエリを書き換えることができます。

```SQL
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey;
```

その元のクエリプランと書き換え後のものは次のようになります。

## 連合の書き換え

### 述語の連合の書き換え

マテリアライズドビューの述語範囲がクエリの述語範囲のサブセットである場合、クエリはUNION演算を使用して書き換えることができます。

たとえば、次のようにマテリアライズドビューを作成します。

```SQL
CREATE MATERIALIZED VIEW agg_mv4
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
WHERE lo_orderkey < 300000000
GROUP BY lo_orderkey;
```

これにより次のクエリが書き換えられます。

```SQL
select 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

その元のクエリプランと書き換え後のものは次のようになります。

## パーティションの連合の書き換え

テーブルのパーティションに基づいたパーティション化されたマテリアライズドビューを作成したとします。クエリでスキャンされる書き換え可能なクエリが最も最近のパーティション範囲のサブセットである場合、クエリはUNION演算を使用して書き換えられます。

たとえば、次のようにマテリアライズドビュー`agg_mv4`を作成します。そのベーステーブル`lineorder`は現在`p1`から`p7`のパーティションを含んでおり、マテリアライズドビューも`p1`から`p7`のパーティションを含んでいます。

```SQL
CREATE MATERIALIZED VIEW agg_mv5
DISTRIBUTED BY hash(lo_orderkey)
PARTITION BY RANGE(lo_orderdate)
REFRESH MANUAL
AS
SELECT 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

もし新しい`p8`というパーティションが`lineorder`に追加された場合、次のクエリはUNION演算を使用して書き換えることができます。

```SQL
SELECT 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

その元のクエリプランと書き換え後のものは次のようになります。

## ビューに基づいたマテリアライズドビューの書き換え

StarRocksは、ビューに基づいてマテリアライズドビューを作成することをサポートしており、ビューに対する後続のクエリは透過的に書き換えることができます。

たとえば、次のようなビューを作成します。

```SQL
CREATE VIEW customer_view1 
AS
SELECT c_custkey, c_name, c_address
FROM customer;

CREATE VIEW lineorder_view1
AS
SELECT lo_orderkey, lo_linenumber, lo_custkey, lo_revenue
FROM lineorder;
```

次に、次のようなビューに基づいたマテリアライズされたビューを作成します。

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name
FROM lineorder_view1 INNER JOIN customer_view1
ON lo_custkey = c_custkey;
```

クエリの書き換え中、`customer_view1`や`lineorder_view1`に対するクエリは自動的に基本テーブルに展開され、透過的にマッチングおよび書き換えされます。

## 外部カタログに基づいたマテリアライズドビューの書き換え

StarRocksは、Hiveカタログ、Hudiカタログ、Icebergカタログ上に非同期マテリアライズドビューを構築し、これらを使用してクエリを透過的に書き換えることをサポートしています。外部カタログに基づくマテリアライズドビューは、クエリの書き換えについてほとんどの機能をサポートしていますが、いくつかの制限があります。

- Hudi、Iceberg、またはJDBCカタログベースのマテリアライズドビューは、連合の書き換えをサポートしていません。
- Hudi、Iceberg、またはJDBCカタログベースのマテリアライズドビューは、ビューデルタジョインの書き換えをサポートしていません。
- Hudi、Iceberg、またはJDBCカタログベースのマテリアライズドビューは、パーティションの増分更新をサポートしていません。

## クエリの書き換えを構成する

非同期マテリアライズドビューのクエリの書き換えは、以下のセッション変数を使用して構成することができます。

| **変数**                                      | **デフォルト** | **説明**                                                      |
| ------------------------------------------- | ----------- | ------------------------------------------------------------ |
| enable_materialized_view_union_rewrite      | true        | マテリアライズドビューのUNIONクエリ書き換えを有効にするかどうかを制御するブール値。 |
| enable_rule_based_materialized_view_rewrite | true        | 単一テーブルクエリ書き換えで使用されるブール値。 |
| nested_mv_rewrite_max_level                 | 3           | クエリ書き換えに使用されるネストされたマテリアライズドビューの最大レベル。タイプ：INT。範囲：[1、+∞)。 `1`の値は、他のマテリアライズドビュー上に作成されたマテリアライズドビューがクエリの書き換えに使用されないことを示します。 |

## クエリが書き換えられたかどうかを確認する

EXPLAINステートメントを使用してクエリのクエリプランを表示することで、クエリが書き換えられたかどうかを確認することができます。`OlapScanNode`セクションの`TABLE`フィールドが対応するマテリアライズドビューの名前を示している場合、クエリはマテリアライズドビューを基に書き換えられたことを意味します。
```
20 rows in set (0.01 sec)
```

## クエリ書き換えの無効化

デフォルトでは、StarRocksはデフォルトカタログに基づいて作成された非同期マテリアライズドビューに対してクエリの書き換えを有効にします。セッション変数 `enable_materialized_view_rewrite` を `false` に設定することで、この機能を無効にすることができます。

外部カタログに基づいて作成された非同期マテリアライズドビューに対しては、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用して、マテリアライズドビュープロパティ `force_external_table_query_rewrite` を `false` に設定することで、この機能を無効にすることができます。

## 制限事項

マテリアライズドビューに基づくクエリの書き換えに関して、StarRocksには現在、次の制限事項があります。

- StarRocksは、rand、random、uuid、sleep を含む非決定論的な関数を使用したクエリの書き換えをサポートしていません。
- StarRocksはウィンドウ関数を使用したクエリの書き換えをサポートしていません。
- LIMIT、ORDER BY、UNION、EXCEPT、INTERSECT、MINUS、GROUPING SETS、WITH CUBE、または WITH ROLLUP を含むステートメントで定義されたマテリアライズドビューは、クエリの書き換えに使用することはできません。
- 外部カタログに基づくベーステーブルとそれに構築されたマテリアライズドビュー間のクエリ結果の強力な整合性は保証されません。
- JDBCカタログで作成されたベーステーブルに対する非同期マテリアライズドビューは、クエリの書き換えをサポートしていません。
```
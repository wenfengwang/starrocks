---
displayed_sidebar: Chinese
---

# 物化ビューによるクエリ改写

本文では、StarRocks の非同期物化ビューを利用してクエリを改写し、加速する方法について説明します。

## 概要

StarRocks の非同期物化ビューは、主流の SPJG（select-project-join-group-by）パターンに基づく透明なクエリ改写アルゴリズムを採用しています。クエリ文を変更することなく、StarRocks は基本テーブル上のクエリを物化ビュー上のクエリに自動的に改写することができます。予め計算された結果を含む物化ビューは、計算コストを大幅に削減し、クエリの実行を加速するのに役立ちます。

非同期物化ビューに基づくクエリ改写機能は、以下のシナリオで特に有用です：

- **指標のプリアグリゲーション**

  高次元データを処理する必要がある場合、物化ビューを使用してプリアグリゲーション指標層を作成できます。

- **ワイドテーブルの Join**

  物化ビューを使用すると、複雑なシナリオでワイドテーブルの Join を含むクエリを透明に加速できます。

- **データレイクアクセラレーション**

  External Catalog ベースの物化ビューを構築することで、データレイク内のデータに対するクエリを容易に加速できます。

> **注意**
>
> JDBC Catalog ベースのテーブルで構築された非同期物化ビューは、現在クエリ改写をサポートしていません。

### 機能の特徴

StarRocks の非同期物化ビューによる自動クエリ改写機能には、以下の特徴があります：

- **強力なデータ一貫性**：基本テーブルが StarRocks の内部テーブルである場合、物化ビューを通じたクエリ改写で得られる結果が、基本テーブルを直接クエリした結果と一致することを StarRocks は保証します。
- **Staleness rewrite**：StarRocks は Staleness rewrite をサポートしており、データの変更が頻繁に発生する状況に対応するため、ある程度のデータの古さを許容することができます。
- **複数テーブルの Join**：StarRocks の非同期物化ビューは、Inner Join、Cross Join、Left Outer Join、Full Outer Join、Right Outer Join、Semi Join、Anti Join など、さまざまなタイプの Join をサポートしており、View Delta Join や Join 派生改写など、複雑な Join シナリオを含む大規模なワイドテーブルのクエリを加速することができます。
- **集約の改写**：StarRocks は集約操作を含むクエリを改写して、レポートのパフォーマンスを向上させることができます。
- **ネストされた物化ビュー**：StarRocks はネストされた物化ビューに基づいて複雑なクエリを改写し、改写可能なクエリの範囲を拡大します。
- **Union の改写**：Union 改写機能を物化ビューのパーティションの生存時間（TTL）と組み合わせることで、コールドデータとホットデータを分離し、物化ビューからホットデータをクエリし、基本テーブルから履歴データをクエリすることができます。
- **ビューに基づいて物化ビューを構築**：ビューベースのモデリングシナリオでクエリを加速することができます。
- **External Catalog に基づいて物化ビューを構築**：この機能を使用して、データレイク内のクエリを加速できます。
- **複雑な式の改写**：式内で関数呼び出しや算術演算をサポートし、複雑な分析や計算ニーズに応えます。

これらの特徴については、以下の各節で詳しく説明します。

## Join の改写

StarRocks は、Inner Join、Cross Join、Left Outer Join、Full Outer Join、Right Outer Join、Semi Join、Anti Join など、さまざまなタイプの Join を含むクエリを改写することをサポートしています。以下の例は、Join クエリの改写を示しています。以下の基本テーブルを作成します：

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

上記の基本テーブルに基づいて、以下の物化ビューを作成します：

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY HASH(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, lo_partkey, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

この物化ビューは、以下のクエリを改写することができます：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

元のクエリプランと改写後のプランは以下の通りです：

![Rewrite-1](../assets/Rewrite-1.png)

StarRocks は、算術演算、文字列関数、日付関数、CASE WHEN 式、OR などの複雑な式を含む Join クエリの改写もサポートしています。例えば、上記の物化ビューは以下のクエリを改写することができます：

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

通常のシナリオに加えて、StarRocks はより複雑なシナリオでの Join クエリの改写もサポートしています。

### Query Delta Join の改写

Query Delta Join は、クエリ中の Join テーブルが物化ビュー中の Join テーブルのスーパーセットである場合を指します。例えば、以下のクエリでは `lineorder`、`customer`、`part` のテーブルを Join しています。物化ビュー `join_mv1` が `lineorder` と `customer` の Join のみを含んでいる場合、StarRocks は `join_mv1` を使用してクエリを改写することができます。

例：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address, p_name
FROM
    lineorder INNER JOIN customer ON lo_custkey = c_custkey
    INNER JOIN part ON lo_partkey = p_partkey;
```

元のクエリプランと改写後のプランは以下の通りです：

![Rewrite-2](../assets/Rewrite-2.png)

### View Delta Join の改写

View Delta Join は、クエリ中の Join テーブルが物化ビュー中の Join テーブルのサブセットである場合を指します。通常、ワイドテーブルを扱うシナリオでこの機能を使用します。例えば、Star Schema Benchmark (SSB) のコンテキストでは、すべてのテーブルを Join する物化ビューを作成することで、クエリのパフォーマンスを向上させることができます。物化ビューを通じてクエリを透明に改写した後、複数テーブルの Join クエリのパフォーマンスは、対応するワイドテーブルをクエリした場合と同等のレベルに達することがテストで確認されています。

View Delta Join の改写を有効にするには、物化ビューにクエリには存在しない 1:1 の Cardinality Preservation Join を含める必要があります。以下の制約条件を満たす 9 種類の Join は、Cardinality Preservation Join と見なされ、View Delta Join の改写に使用できます：

![Rewrite-3](../assets/Rewrite-3.png)

SSB のテストを例に、以下の基本テーブルを作成します：

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
"unique_constraints" = "c_custkey"   -- 指定唯一キー。
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
  d_weeknuminyear    INT(11)       NOT NULL,
  d_sellingseason    VARCHAR(14)   NOT NULL,
  d_lastdayinweekfl  INT(11)       NOT NULL,
  d_lastdayinmonthfl INT(11)       NOT NULL,
  d_holidayfl        INT(11)       NOT NULL,
  d_weekdayfl        INT(11)       NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(d_datekey)
DISTRIBUTED BY HASH(d_datekey) BUCKETS 1
PROPERTIES (
"unique_constraints" = "d_datekey"   -- 指定唯一キー。
);

CREATE TABLE supplier (
  s_suppkey          INT(11)       NOT NULL,
  s_name             VARCHAR(26)   NOT NULL,
  s_address          VARCHAR(26)   NOT NULL,
  s_city             VARCHAR(11)   NOT NULL,
  s_nation           VARCHAR(16)   NOT NULL,
  s_region           VARCHAR(13)   NOT NULL,
  s_phone            VARCHAR(16)   NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(s_suppkey)
DISTRIBUTED BY HASH(s_suppkey) BUCKETS 12
PROPERTIES (
"unique_constraints" = "s_suppkey"   -- 指定唯一キー。
);

CREATE TABLE part (
  p_partkey          INT(11)       NOT NULL,
  p_name             VARCHAR(23)   NOT NULL,
  p_mfgr             VARCHAR(7)    NOT NULL,
  p_category         VARCHAR(8)    NOT NULL,
  p_brand            VARCHAR(10)   NOT NULL,
  p_color            VARCHAR(12)   NOT NULL,
  p_type             VARCHAR(26)   NOT NULL,
  p_size             TINYINT(11)   NOT NULL,
  p_container        VARCHAR(11)   NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(p_partkey)
DISTRIBUTED BY HASH(p_partkey) BUCKETS 12
PROPERTIES (
"unique_constraints" = "p_partkey"   -- 指定唯一キー。
);

CREATE TABLE lineorder (
  lo_orderdate       DATE          NOT NULL, -- NOT NULL として指定。
  lo_orderkey        INT(11)       NOT NULL,
  lo_linenumber      TINYINT       NOT NULL,
  lo_custkey         INT(11)       NOT NULL, -- NOT NULL として指定。
  lo_partkey         INT(11)       NOT NULL, -- NOT NULL として指定。
  lo_suppkey         INT(11)       NOT NULL, -- NOT NULL として指定。
  lo_orderpriority   VARCHAR(100)  NOT NULL,
  lo_shippriority    TINYINT       NOT NULL,
  lo_quantity        TINYINT       NOT NULL,
  lo_extendedprice   INT(11)       NOT NULL,
  lo_ordtotalprice   INT(11)       NOT NULL,
  lo_discount        TINYINT       NOT NULL,
  lo_revenue         INT(11)       NOT NULL,
  lo_supplycost      INT(11)       NOT NULL,
  lo_tax             TINYINT       NOT NULL,
  lo_commitdate      DATE          NOT NULL,
  lo_shipmode        VARCHAR(100)  NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(lo_orderdate, lo_orderkey)
PARTITION BY RANGE(lo_orderdate)
(PARTITION p1 VALUES [("0000-01-01"), ("1993-01-01")),
PARTITION p2 VALUES [("1993-01-01"), ("1994-01-01")),
PARTITION p3 VALUES [("1994-01-01"), ("1995-01-01")),
PARTITION p4 VALUES [("1995-01-01"), ("1996-01-01")),
PARTITION p5 VALUES [("1996-01-01"), ("1997-01-01")),
PARTITION p6 VALUES [("1997-01-01"), ("1998-01-01")),
PARTITION p7 VALUES [("1998-01-01"), ("1999-01-01"]))
DISTRIBUTED BY HASH(lo_orderkey) BUCKETS 48
PROPERTIES (
"foreign_key_constraints" = "
    (lo_custkey) REFERENCES customer(c_custkey);
    (lo_partkey) REFERENCES part(p_partkey);
    (lo_suppkey) REFERENCES supplier(s_suppkey)" -- 指定外键制約。
);
```

`lineorder`、`customer`、`supplier`、`part`、`dates` テーブルのマテリアライズドビュー `lineorder_flat_mv` を作成する:

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH MANUAL
PROPERTIES (
    "partition_refresh_number"="1"
)
AS SELECT /*+ SET_VAR(query_timeout = 7200) */     -- リフレッシュタイムアウトを設定。
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

SSB Q2.1 は4つのテーブルを結合するクエリですが、マテリアライズドビュー `lineorder_flat_mv` と比較すると `customer` テーブルが欠けています。`lineorder_flat_mv` では、`lineorder INNER JOIN customer` は本質的に Cardinality Preservation Join です。したがって、論理的にはこの結合を省略してもクエリ結果に影響しません。従って、Q2.1 は `lineorder_flat_mv` を使用して書き換えることができます。

SSB Q2.1:

```SQL
SELECT sum(lo_revenue) AS lo_revenue, d_year, p_brand
FROM lineorder
JOIN dates ON lo_orderdate = d_datekey
JOIN part ON lo_partkey = p_partkey
JOIN supplier ON lo_suppkey = s_suppkey
WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
GROUP BY d_year, p_brand
ORDER BY d_year, p_brand;
```

元のクエリプランと書き換え後のプランは以下の通りです：

![Rewrite-4](../assets/Rewrite-4.png)

同様に、SSBの他のクエリも `lineorder_flat_mv` を使用して透過的に書き換えることで、クエリパフォーマンスを最適化できます。

### Join 派生改写

Join 派生とは、物化ビューとクエリの Join の種類が一致しないが、物化ビューの Join 結果がクエリの Join 結果を含む場合を指します。現在、3つ以上のテーブルの Join と2つのテーブルの Join の2つのシナリオがサポートされています。

- **シナリオ1：3つ以上のテーブルの Join**

  物化ビューが `t1` と `t2` の間の Left Outer Join と、`t2` と `t3` の間の Inner Join を含むとします。両方の Join の条件には `t2` の列が含まれています。

  一方、クエリは `t1` と `t2` の間の Inner Join と、`t2` と `t3` の間の Inner Join を含んでいます。両方の Join の条件には `t2` の列が含まれています。

  この場合、上記のクエリは物化ビューを使用して書き換えることができます。これは、物化ビューでは最初に Left Outer Join が実行され、次に Inner Join が実行されるためです。Left Outer Join によって生成された右側のテーブルには一致する結果がなく（つまり、右側のテーブルの列が NULL）、Inner Join の実行中にフィルタリングされます。したがって、物化ビューとクエリの論理は等価であり、クエリを書き換えることができます。

  例：

  物化ビュー `join_mv5` を作成する：

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

  `join_mv5` は以下のクエリを書き換えることができます：

  ```SQL
  SELECT lo_orderkey, lo_orderdate, lo_linenumber, lo_revenue, c_custkey, c_address, p_name
  FROM customer INNER JOIN lineorder
  ON c_custkey = lo_custkey
  INNER JOIN part
  ON p_partkey = lo_partkey;
  ```

  元のクエリプランと書き換え後のプランは以下の通りです：

  ![Rewrite-5](../assets/Rewrite-5.png)

  同様に、物化ビューが `t1 INNER JOIN t2 INNER JOIN t3` として定義されている場合、クエリが `LEFT OUTER JOIN t2 INNER JOIN t3` であっても、クエリは書き換えることができます。また、3つ以上のテーブルが関与する場合でも、上記の書き換え能力があります。

- **シナリオ2：2つのテーブルの Join**

  2つのテーブルの Join の派生改写は、以下のいくつかのサブシナリオをサポートしています：

  ![Rewrite-6](../assets/Rewrite-6.png)

  シナリオ1から9では、書き換え結果にフィルタリング述語を補償する必要があり、意味的な等価性を保証するためです。例えば、以下の物化ビューを作成する：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv3
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  すると `join_mv3` は以下のクエリを書き換えることができ、クエリ結果に `c_custkey IS NOT NULL` の述語を補償する必要があります：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  元のクエリプランと書き換え後のプランは以下の通りです：

  ![Rewrite-7](../assets/Rewrite-7.png)

  シナリオ10では、Left Outer Join のクエリに右側のテーブルの `IS NOT NULL` のフィルタリング述語が含まれている必要があります。例えば `=`、`<>`、`>`、`<`、`<=`、`>=`、`LIKE`、`IN`、`NOT LIKE`、`NOT IN` などです。例えば、以下の物化ビューを作成する：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv4
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  すると `join_mv4` は以下のクエリを書き換えることができ、`customer.c_address = "Sb4gxKs7"` が `IS NOT NULL` の述語として機能します：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey
  WHERE customer.c_address = "Sb4gxKs7";
  ```

  元のクエリプランと書き換え後のプランは以下の通りです：

  ![Rewrite-8](../assets/Rewrite-8.png)

## 聚合改写

StarRocks の非同期マテリアライズドビューによる多表聚合クエリの改写は、bitmap_union、hll_union、percentile_union などを含むすべての聚合関数をサポートしています。例えば、以下のマテリアライズドビューを作成する：

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

このマテリアライズドビューは以下のクエリを改写することができます：

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

元のクエリプランと書き換え後のプランは以下の通りです：

![Rewrite-9](../assets/Rewrite-9.png)

以下のセクションでは、聚合改写機能が利用可能なシナリオについて詳しく説明します。

### 聚合上卷改写


StarRocks は、集約ロールアップによるクエリの書き換えをサポートしています。つまり、`GROUP BY a,b` 句で作成された非同期マテリアライズドビューを使用して、`GROUP BY a` 句を含む集約クエリを書き換えることができます。例えば、`agg_mv1` は以下のクエリを書き換えることができます：

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

その元のクエリプランと書き換え後のプランは以下の通りです：

![Rewrite-10](../assets/Rewrite-10.png)

> **説明**
>
> 現在、grouping set、grouping set with rollup、grouping set with cube の書き換えはサポートされていません。

一部の集約関数のみが集約ロールアップクエリの書き換えをサポートしています。以下の表は、元のクエリの集約関数とマテリアライズドビューの構築に使用される集約関数との対応関係を示しています。ビジネスシナリオに応じて、適切な集約関数を選択してマテリアライズドビューを構築できます。

| **元のクエリの集約関数**                                      | **Aggregate Rollup 対応のマテリアライズドビュー構築集約関数**                 |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| sum                                                    | sum                                                          |
| count                                                  | count                                                        |
| min                                                    | min                                                          |
| max                                                    | max                                                          |
| avg                                                    | sum / count                                                  |
| bitmap_union, bitmap_union_count, count(distinct)      | bitmap_union                                                 |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                                                    |
| percentile_approx, percentile_union                    | percentile_union                                             |

GROUP BY 列がない DISTINCT 集約は集約ロールアップクエリの書き換えには使用できません。しかし、StarRocks v3.1 からは、集約ロールアップが DISTINCT 集約関数のクエリに対応しており、GROUP BY 列がなくても、等価な述語があれば、関連するマテリアライズドビューによってクエリが書き換えられる可能性があります。これは、StarRocks が等価な述語を GROUP BY の定数式に変換できるためです。

以下の例では、StarRocks はマテリアライズドビュー `order_agg_mv1` を使用して対応するクエリ Query を書き換えることができます：

```SQL
CREATE MATERIALIZED VIEW order_agg_mv1
DISTRIBUTED BY HASH(`order_id`) BUCKETS 12
REFRESH ASYNC START('2022-09-01 10:00:00') EVERY (interval 1 day)
AS
SELECT
    order_date,
    count(distinct client_id) 
FROM order_list 
GROUP BY order_date;


-- Query
SELECT
    order_date,
    count(distinct client_id) 
FROM order_list WHERE order_date='2023-07-03';
```

### COUNT DISTINCT の書き換え

StarRocks は、COUNT DISTINCT を BITMAP 型の計算に書き換えて、マテリアライズドビューを使用した高性能で正確な重複排除を実現することをサポートしています。例えば、以下のマテリアライズドビューを作成します：

```SQL
CREATE MATERIALIZED VIEW distinct_mv
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, bitmap_union(to_bitmap(lo_custkey)) AS distinct_customer
FROM lineorder
GROUP BY lo_orderkey;
```

このマテリアライズドビューは以下のクエリを書き換えることができます：

```SQL
SELECT lo_orderkey, count(distinct lo_custkey) 
FROM lineorder 
GROUP BY lo_orderkey;
```

## ネストされたマテリアライズドビューの書き換え

StarRocks は、ネストされたマテリアライズドビューを使用したクエリの書き換えをサポートしています。例えば、以下のマテリアライズドビュー `join_mv2`、`agg_mv2`、`agg_mv3` を作成します：

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

その関係は以下の通りです：

![Rewrite-11](../assets/Rewrite-11.png)

`agg_mv3` は以下のクエリを書き換えることができます：

```SQL
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey;
```

その元のクエリプランと書き換え後のプランは以下の通りです：

![Rewrite-12](../assets/Rewrite-12.png)

## Union の書き換え

### 谓词 Union の書き換え

物化ビューの述語範囲がクエリの述語範囲のサブセットである場合、UNION 操作を使用してクエリを書き換えることができます。

例えば、以下のマテリアライズドビューを作成します：

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

このマテリアライズドビューは以下のクエリを書き換えることができます：

```SQL
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

その元のクエリプランと書き換え後のプランは以下の通りです：

![Rewrite-13](../assets/Rewrite-13.png)

ここで、`agg_mv4` は `lo_orderkey < 300000000` のデータを含み、`lo_orderkey >= 300000000` のデータは `lineorder` テーブルから直接取得されます。最終的には、Union 操作を通じて集約され、最終結果が得られます。

### 分区 Union の書き換え

分区テーブルに基づいて分区マテリアライズドビューが作成されたと仮定します。クエリがスキャンする分区範囲がマテリアライズドビューの最新分区範囲のスーパーセットである場合、クエリは UNION で書き換えることができます。

例えば、以下のマテリアライズドビュー `agg_mv5` があります。基本テーブル `lineorder` は現在、分区 `p1` から `p7` を含んでおり、マテリアライズドビューも分区 `p1` から `p7` を含んでいます。

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

もし `lineorder` に新しい分区 `p8` が追加され、その範囲が `[("19990101"), ("20000101"))` である場合、以下のクエリは UNION で書き換えることができます：

```SQL
SELECT 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

その元のクエリプランと書き換え後のプランは以下の通りです：

![Rewrite-14](../assets/Rewrite-14.png)

上記のように、`agg_mv5` は分区 `p1` から `p7` までのデータを含み、分区 `p8` のデータは `lineorder` から取得されます。最後に、これらのデータは UNION 操作で結合されます。

## ビューに基づいたマテリアライズドビューの構築

StarRocks は、ビューに基づいてマテリアライズドビューを作成することをサポートしています。後続のビューに対するクエリは透過的に書き換えられます。

例えば、以下のビューを作成します：

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

上記のビューに基づいてマテリアライズドビューを作成します：

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name
FROM lineorder_view1 INNER JOIN customer_view1
ON lo_custkey = c_custkey;
```

クエリの書き換えプロセスでは、`customer_view1` および `lineorder_view1` に対するクエリは自動的に基本テーブルに展開され、その後透過的に書き換えが行われます。

## External Catalog に基づいたマテリアライズドビューの構築

StarRocks は、Hive Catalog、Hudi Catalog、Iceberg Catalog などの外部データソースに基づいた非同期マテリアライズドビューの構築をサポートし、クエリを透過的に書き換えることができます。External Catalog に基づいたマテリアライズドビューは、ほとんどのクエリの書き換え機能をサポートしていますが、以下の制限があります：

- Hudi、Iceberg、JDBC Catalog に基づいて作成されたマテリアライズドビューは Union の書き換えをサポートしていません。
- Hudi、Iceberg、JDBC Catalog に基づいて作成されたマテリアライズドビューは View Delta Join の書き換えをサポートしていません。
- Hudi、Iceberg、JDBC Catalog に基づいて作成されたマテリアライズドビューは分区の増分リフレッシュをサポートしていません。

## マテリアライズドビューのクエリ書き換えの設定

以下のセッション変数を使用して、非同期マテリアライズドビューのクエリ書き換えを設定できます。

| **変数**                                    | **デフォルト値** | **説明**                                                     |
| ------------------------------------------- | ---------- | ------------------------------------------------------------ |
| enable_materialized_view_union_rewrite | true | マテリアライズドビューの Union 書き換えを有効にするかどうか。  |
| enable_rule_based_materialized_view_rewrite | true | ルールベースのマテリアライズドビューのクエリ書き換え機能を有効にするかどうか。主に単一テーブルのクエリ書き換えを処理するために使用されます。 |
| nested_mv_rewrite_max_level | 3 | クエリ書き換えに使用できるネストされたマテリアライズドビューの最大レベル。型：INT。範囲：[1, +∞)。`1` に設定すると、基本テーブルに基づいて作成されたマテリアライズドビューのみを使用してクエリの書き換えが可能です。 |

## クエリ書き換えが有効かどうかの確認

EXPLAIN ステートメントを使用して、対応する Query Plan を確認できます。`OlapScanNode` セクションの `TABLE` が対応する非同期マテリアライズドビューの名前であれば、そのクエリが非同期マテリアライズドビューに基づいて書き換えられたことを意味します。

```Plain
mysql> EXPLAIN SELECT 
    order_id, sum(goods.price) AS total 
    FROM order_list INNER JOIN goods 
    ON goods.item_id1 = order_list.item_id2 
    GROUP BY order_id;
+------------------------------------+
| Explain String                     |
+------------------------------------+
| PLAN FRAGMENT 0                    |
|  OUTPUT EXPRS:1: order_id | 8: sum |
|   PARTITION: RANDOM                |
|                                    |
|   RESULT SINK                      |
|                                    |
|   1:Project                        |
|   |  <slot 1> : 9: order_id        |
|   |  <slot 8> : 10: total          |
|   |                                |
|   0:OlapScanNode                   |
|      TABLE: order_mv               |
|      PREAGGREGATION: ON            |
|      partitions=1/1                |
|      rollup: order_mv              |
|      tabletRatio=0/12              |
|      tabletList=                   |
|      cardinality=3                 |
|      avgRowSize=4.0                |
|      numNodes=0                    |
+------------------------------------+
20 rows in set (0.01 sec)
```

## クエリ書き換えの無効化

StarRocks はデフォルトで Default Catalog に基づいて作成された非同期マテリアライズドビューのクエリ書き換えを有効にしています。セッション変数 `enable_materialized_view_rewrite` を `false` に設定することで、この機能を無効にできます。

External Catalog を基に作成された非同期マテリアライズドビューについては、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用してマテリアライズドビューのプロパティ `force_external_table_query_rewrite` を `false` に設定することで、この機能を無効にすることができます。

## 制限事項

マテリアライズドビューのクエリ改写機能に関して、StarRocks には以下の制限があります：

- StarRocks は非決定論的関数の改写をサポートしていません。これには rand、random、uuid、sleep が含まれます。
- StarRocks はウィンドウ関数の改写をサポートしていません。
- マテリアライズドビューの定義に LIMIT、ORDER BY、UNION、EXCEPT、INTERSECT、MINUS、GROUPING SETS、WITH CUBE、WITH ROLLUP が含まれている場合、改写には使用できません。
- External Catalog を基にしたマテリアライズドビューは、クエリ結果の強い一貫性を保証しません。
- JDBC Catalog を基にした表で構築された非同期マテリアライズドビューは、現在クエリ改写をサポートしていません。

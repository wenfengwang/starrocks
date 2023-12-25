---
displayed_sidebar: English
---

# マテリアライズドビューを利用したクエリの書き換え

このトピックでは、StarRocksの非同期マテリアライズドビューを活用してクエリを書き換え、高速化する方法について説明します。

## 概要

StarRocksの非同期マテリアライズドビューは、SPJG（select-project-join-group-by）形式に基づいた広く採用されている透過的なクエリ書き換えアルゴリズムを使用します。クエリステートメントを変更することなく、ベーステーブルに対するクエリを事前に計算された結果を含む対応するマテリアライズドビューに対するクエリに自動的に書き換えることができます。結果として、マテリアライズドビューは計算コストを大幅に削減し、クエリ実行を大幅に加速するのに役立ちます。

非同期マテリアライズドビューに基づくクエリ書き換え機能は、以下のシナリオで特に有用です：

- **メトリクスの事前集計**

  高次元データを扱う場合、マテリアライズドビューを使用して事前集計されたメトリクスレイヤーを作成できます。

- **幅広いテーブルの結合**

  マテリアライズドビューを使用すると、複雑なシナリオで複数の大規模な幅広いテーブルの結合を透過的に加速することができます。

- **データレイクでのクエリ加速**

  外部カタログベースのマテリアライズドビューを構築することで、データレイク内のデータに対するクエリを容易に加速できます。

  > **注記**
  >
  > JDBCカタログ内のベーステーブルで作成された非同期マテリアライズドビューは、クエリ書き換えをサポートしていません。

### 特徴

StarRocksの非同期マテリアライズドビューに基づく自動クエリ書き換えには、以下の特徴があります：

- **強固なデータ整合性**: ベーステーブルがネイティブテーブルである場合、StarRocksはマテリアライズドビューに基づくクエリ書き換えを通じて得られた結果が、ベーステーブルに対する直接クエリから返される結果と一致することを保証します。
- **陳腐化による書き換え**: StarRocksは陳腐化による書き換えをサポートし、データの変更が頻繁に行われるシナリオにおいて、ある程度のデータの古さを許容することができます。
- **複数テーブル結合**: StarRocksの非同期マテリアライズドビューは、View Delta JoinsやDerivable Joinsなどの複雑な結合シナリオを含むさまざまなタイプの結合をサポートし、大規模な幅広いテーブルを含むシナリオでのクエリを加速できます。
- **集約による書き換え**: StarRocksは集約を含むクエリを書き換えてレポートのパフォーマンスを向上させることができます。
- **ネストされたマテリアライズドビュー**: StarRocksはネストされたマテリアライズドビューに基づく複雑なクエリの書き換えをサポートし、書き換え可能なクエリの範囲を拡大します。
- **ユニオンによる書き換え**: ユニオンの書き換え機能をマテリアライズドビューのパーティションのTTL（Time-to-Live）と組み合わせることで、ホットデータとコールドデータの分離を実現し、マテリアライズドビューからホットデータを、ベーステーブルからは履歴データをクエリすることができます。
- **ビュー上のマテリアライズドビュー**: ビューに基づくデータモデリングを使用したシナリオでのクエリを加速できます。
- **外部カタログ上のマテリアライズドビュー**: データレイクでのクエリを加速できます。
- **複雑な式の書き換え**: 関数呼び出しや算術演算を含む複雑な式を処理でき、高度な分析や計算要件に対応します。

これらの特徴については、後続のセクションで詳しく説明します。

## 結合クエリの書き換え

StarRocksは、Inner Join、Cross Join、Left Outer Join、Full Outer Join、Right Outer Join、Semi Join、Anti Joinなど、さまざまなタイプの結合を含むクエリの書き換えをサポートしています。

以下は結合を使用したクエリの書き換えの例です。次のように2つのベーステーブルを作成します：

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

上記のベーステーブルを使用して、次のようにマテリアライズドビューを作成できます：

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY HASH(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, lo_partkey, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

このマテリアライズドビューは、以下のクエリを書き換えることができます：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

![Rewrite-1](../assets/Rewrite-1.png)

StarRocksは、算術演算、文字列関数、日付関数、CASE WHEN式、OR述語などの複雑な式を含む結合クエリの書き換えもサポートしています。例えば、上記のマテリアライズドビューは、以下のクエリを書き換えることができます：

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

従来のシナリオに加えて、StarRocksはより複雑なシナリオでの結合クエリの書き換えをさらにサポートしています。

### クエリデルタ結合の書き換え

クエリデルタ結合は、クエリで結合されたテーブルがマテリアライズドビューで結合されたテーブルのスーパーセットであるシナリオを指します。例えば、`lineorder`、`customer`、`part`の3つのテーブルの結合を含む以下のクエリを考えてみましょう。マテリアライズドビュー`join_mv1`が`lineorder`と`customer`の結合のみを含んでいる場合、StarRocksは`join_mv1`を使用してクエリを書き換えることができます。

例：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address, p_name
FROM
    lineorder INNER JOIN customer ON lo_custkey = c_custkey
    INNER JOIN part ON lo_partkey = p_partkey;
```

元のクエリプランと書き換え後のクエリプランは以下の通りです：

![Rewrite-2](../assets/Rewrite-2.png)

### ビューデルタ結合の書き換え

ビューデルタ結合は、クエリで結合されるテーブルがマテリアライズドビューで結合されるテーブルのサブセットであるシナリオを指します。この機能は通常、大規模な幅広いテーブルを使用するシナリオで使用されます。例えば、スタースキーマベンチマーク（SSB）のコンテキストで、すべてのテーブルを結合するマテリアライズドビューを作成してクエリパフォーマンスを向上させることができます。テストにより、マテリアライズドビューを使用してクエリを透過的に書き換えた後の複数テーブル結合のクエリパフォーマンスが、対応する大規模な幅広いテーブルをクエリする場合と同等のパフォーマンスを達成できることが確認されています。

ビューのデルタ結合の書き換えを実行するには、マテリアライズドビューにクエリに存在しない1:1のカーディナリティ保持結合が含まれている必要があります。カーディナリティ保持結合と見なされる9種類の結合を以下に示し、いずれか一つを満たすことでビューのデルタ結合の書き換えが可能になります：

![Rewrite-3](../assets/Rewrite-3.png)

SSBテストを例に、以下のベーステーブルを作成します：

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
"unique_constraints" = "d_datekey"   -- 一意の制約を指定します。
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
"unique_constraints" = "s_suppkey"   -- 一意の制約を指定します。
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
"unique_constraints" = "p_partkey"   -- 一意の制約を指定します。
);

CREATE TABLE lineorder (
  lo_orderdate       DATE          NOT NULL, -- NOT NULLとして指定します。
  lo_orderkey        INT(11)       NOT NULL,
  lo_linenumber      TINYINT       NOT NULL,
  lo_custkey         INT(11)       NOT NULL, -- NOT NULLとして指定します。
  lo_partkey         INT(11)       NOT NULL, -- NOT NULLとして指定します。
  lo_suppkey         INT(11)       NOT NULL, -- NOT NULLとして指定します。
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
DUPLICATE KEY(lo_orderdate,lo_orderkey)
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
    (lo_suppkey) REFERENCES supplier(s_suppkey)" -- 外部キーを指定します。
);
```

マテリアライズドビュー`lineorder_flat_mv`を作成し、`lineorder`、`customer`、`supplier`、`part`、`dates`を結合します：

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY RANGE(LO_ORDERDATE)
REFRESH MANUAL
PROPERTIES (
    "partition_refresh_number"="1"
)
AS SELECT /*+ SET_VAR(query_timeout = 7200) */     -- リフレッシュ操作のタイムアウトを設定します。
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

SSB Q2.1では4つのテーブルを結合しますが、マテリアライズドビュー`lineorder_flat_mv`に比べて`customer`テーブルが欠けています。`lineorder_flat_mv`において、`lineorder`と`customer`の結合は本質的にカーディナリティ保持結合です。したがって、論理的には、この結合を削除してもクエリ結果に影響を与えません。結果として、Q2.1は`lineorder_flat_mv`を使用して書き換えることができます。

SSB Q2.1：

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

元のクエリプランと書き換え後のクエリプランは以下の通りです：

![Rewrite-4](../assets/Rewrite-4.png)

同様に、SSB内の他のクエリも`lineorder_flat_mv`を使用して透過的に書き換えることができ、クエリパフォーマンスが最適化されます。

### 結合導出性の書き換え

結合導出性とは、マテリアライズドビューとクエリの結合タイプが一致していないが、マテリアライズドビューの結合結果がクエリの結合結果を含むシナリオを指します。現在、3つ以上のテーブルを結合するシナリオと2つのテーブルを結合するシナリオの2つがサポートされています。

- **シナリオ1：3つ以上のテーブルを結合する**

  マテリアライズドビューには、`t1`と`t2`の間のLeft Outer Joinと、`t2`と`t3`の間のInner Joinが含まれているとします。両方の結合で結合条件には`t2`のカラムが含まれています。

  一方でクエリには、`t1`と`t2`の間のInner Joinと、`t2`と`t3`の間のInner Joinが含まれています。両方の結合で結合条件には`t2`のカラムが含まれています。
  この場合、クエリはマテリアライズドビューを使用して書き換えることができます。これは、マテリアライズドビューでは、Left Outer Joinが最初に実行され、その後にInner Joinが実行されるためです。Left Outer Joinによって生成された右側のテーブルには、一致する結果がないため（つまり、右側のテーブルのカラムはNULLです）。これらの結果は、その後、Inner Join中にフィルタリングされます。したがって、マテリアライズドビューとクエリのロジックは同等であり、クエリを書き換えることができます。

  例：

  マテリアライズドビュー `join_mv5` を作成します：

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

  元のクエリプランと書き換え後のクエリプランは以下の通りです：

  ![Rewrite-5](../assets/Rewrite-5.png)

  同様に、マテリアライズドビューが `t1 INNER JOIN t2 INNER JOIN t3` として定義され、クエリが `LEFT OUTER JOIN t2 INNER JOIN t3` の場合も、クエリを書き換えることができます。さらに、この書き換え機能は3つ以上のテーブルを含むシナリオにも適用されます。

- **シナリオ二：二つのテーブルを結合する**

  二つのテーブルを結合するJoin Derivability Rewrite機能は、以下の特定のケースをサポートしています：

  ![Rewrite-6](../assets/Rewrite-6.png)

  ケース1から9では、セマンティックな等価性を保証するために、書き換えた結果にフィルタリング述語を追加する必要があります。例えば、次のようにマテリアライズドビューを作成します：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv3
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  次のクエリは `join_mv3` を使用して書き換えることができ、書き換えた結果に述語 `c_custkey IS NOT NULL` が追加されます：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  元のクエリプランと書き換え後のクエリプランは以下の通りです：

  ![Rewrite-7](../assets/Rewrite-7.png)

  ケース10では、Left Outer Joinクエリに右側のテーブルに対するフィルタリング述語 `IS NOT NULL` を含める必要があります。例えば、`=`、`<>`、`>`、`<`、`<=`、`>=`、`LIKE`、`IN`、`NOT LIKE`、`NOT IN` などです。例として、次のようにマテリアライズドビューを作成します：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv4
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  `join_mv4` は次のクエリを書き換えることができますが、`customer.c_address = "Sb4gxKs7"` はフィルタリング述語 `IS NOT NULL` です：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey
  WHERE customer.c_address = "Sb4gxKs7";
  ```

  元のクエリプランと書き換え後のクエリプランは以下の通りです：

  ![Rewrite-8](../assets/Rewrite-8.png)

## 集約の書き換え

StarRocksの非同期マテリアライズドビューは、bitmap_union、hll_union、percentile_unionを含むすべての利用可能な集約関数を使用して、マルチテーブル集約クエリの書き換えをサポートしています。例えば、次のようにマテリアライズドビューを作成します：

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

元のクエリプランと書き換え後のクエリプランは以下の通りです：

![Rewrite-9](../assets/Rewrite-9.png)

次のセクションでは、集約書き換え機能が有用なシナリオについて詳述します。

### 集約ロールアップの書き換え

StarRocksは、集約ロールアップを使用したクエリの書き換えをサポートしており、`GROUP BY a`句を使用して作成された非同期マテリアライズドビューを使用して、`GROUP BY a, b`句を含む集約クエリを書き換えることができます。例えば、`agg_mv1`を使用して次のクエリを書き換えることができます：

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

元のクエリプランと書き換え後のクエリプランは以下の通りです：

![Rewrite-10](../assets/Rewrite-10.png)

> **注記**
>
> 現在、グループ化セット、ロールアップ付きグループ化セット、またはキューブ付きグループ化セットの書き換えはサポートされていません。

特定の集約関数のみが集約ロールアップでのクエリ書き換えをサポートしています。前述の例では、マテリアライズドビュー `order_agg_mv` が `bitmap_union(to_bitmap(client_id))` を使用している場合、StarRocksは集約ロールアップを使用したクエリの書き換えができますが、`count(distinct client_id)` を使用している場合は書き換えができません。

以下の表は、元のクエリの集約関数と、マテリアライズドビューの構築に使用される集約関数の対応を示しています。ビジネスシナリオに応じて、対応する集約関数を選択してマテリアライズドビューを構築できます。

| **元のクエリでサポートされる集約関数**   | **マテリアライズドビューでサポートされる集約ロールアップ関数** |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| sum                                                    | sum                                                          |
| count                                                  | count                                                        |
| min                                                    | min                                                          |
| max                                                    | max                                                          |
| avg                                                    | sum / count                                                  |
| bitmap_union, bitmap_union_count, count(distinct)      | bitmap_union                                                 |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                                                    |
| percentile_approx, percentile_union                    | percentile_union                                             |

GROUP BY列なしでDISTINCT集約は集約ロールアップで書き換えることができません。しかし、StarRocks v3.1以降、集約ロールアップDISTINCT集約関数を含むクエリにGROUP BY列がなく、等価述語がある場合、StarRocksは等価述語をGROUP BY定数式に変換できるため、関連するマテリアライズドビューで書き換えることができます。

以下の例では、StarRocksはマテリアライズドビュー `order_agg_mv1` を使用してクエリを書き換えることができます。

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


-- クエリ
SELECT
    order_date,
    count(distinct client_id) 
FROM order_list WHERE order_date='2023-07-03';
```

### COUNT DISTINCTの書き換え

StarRocksは、COUNT DISTINCT計算をビットマップベースの計算に書き換えることをサポートしており、マテリアライズドビューを使用した高性能で正確な重複排除を可能にします。たとえば、次のようにマテリアライズドビューを作成します。

```SQL
CREATE MATERIALIZED VIEW distinct_mv
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, bitmap_union(to_bitmap(lo_custkey)) AS distinct_customer
FROM lineorder
GROUP BY lo_orderkey;
```

これは次のクエリを書き換えることができます。

```SQL
SELECT lo_orderkey, count(distinct lo_custkey) 
FROM lineorder 
GROUP BY lo_orderkey;
```

## ネストされたマテリアライズドビューの書き換え

StarRocksは、ネストされたマテリアライズドビューを使用したクエリの書き換えをサポートしています。例えば、マテリアライズドビュー `join_mv2`、`agg_mv2`、そして `agg_mv3` を以下のように作成します：

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

それらの関係は以下の通りです：

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

元のクエリプランと書き換え後のクエリプランは以下の通りです：

![Rewrite-12](../assets/Rewrite-12.png)

## ユニオン書き換え

### プレディケートユニオン書き換え

マテリアライズドビューのプレディケート範囲がクエリのプレディケート範囲のサブセットである場合、クエリはUNION操作を使用して書き換えることができます。

例えば、以下のようにマテリアライズドビューを作成します：

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

これは以下のクエリを書き換えることができます：

```SQL
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

元のクエリプランと書き換え後のクエリプランは以下の通りです：

![Rewrite-13](../assets/Rewrite-13.png)

このコンテキストでは、`agg_mv4` には `lo_orderkey < 300000000` のデータが含まれています。`lo_orderkey >= 300000000` のデータはベーステーブル `lineorder` から直接取得されます。最終的に、これら2つのデータセットはUNION操作を使用して結合され、集約して最終結果が得られます。

### パーティションユニオン書き換え

パーティション化されたテーブルに基づいてパーティション化されたマテリアライズドビューを作成したとします。書き換え可能なクエリがスキャンするパーティション範囲が、マテリアライズドビューの最新のパーティション範囲のスーパーセットである場合、クエリはUNION操作を使用して書き換えられます。

例えば、以下のマテリアライズドビュー `agg_mv5` を考えます。そのベーステーブル `lineorder` には現在 `p1` から `p7` までのパーティションが含まれており、マテリアライズドビューにも `p1` から `p7` までのパーティションが含まれています。

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

`lineorder` に新しいパーティション `p8` が追加され、そのパーティション範囲が `[("19990101"), ("20000101"))` である場合、以下のクエリはUNION操作を使用して書き換えることができます：

```SQL
SELECT 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

元のクエリプランと書き換え後のクエリプランは以下の通りです：

![Rewrite-14](../assets/Rewrite-14.png)

上記のように、`agg_mv5` には `p1` から `p7` までのパーティションのデータが含まれており、`p8` のパーティションのデータは `lineorder` から直接クエリされます。最終的に、これら2つのデータセットはUNION操作を使用して結合されます。

## ビューベースのマテリアライズドビュー書き換え

StarRocksは、ビューに基づいたマテリアライズドビューの作成をサポートしており、ビューに対する後続のクエリは透過的に書き換えられます。

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

次に、ビューに基づいて以下のマテリアライズドビューを作成します：

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name
FROM lineorder_view1 INNER JOIN customer_view1
ON lo_custkey = c_custkey;
```

クエリ書き換え時、`customer_view1` および `lineorder_view1` に対するクエリは自動的にベーステーブルに展開され、透過的にマッチングおよび書き換えられます。

## 外部カタログベースのマテリアライズドビュー書き換え

StarRocksは、Hiveカタログ、Hudiカタログ、Icebergカタログに基づいた非同期マテリアライズドビューの構築をサポートし、それらを使用してクエリを透過的に書き換えることができます。外部カタログベースのマテリアライズドビューは、ほとんどのクエリ書き換え機能をサポートしていますが、いくつかの制限があります：

- Hudi、Iceberg、またはJDBCカタログベースのマテリアライズドビューは、ユニオン書き換えをサポートしていません。
- Hudi、Iceberg、またはJDBCカタログベースのマテリアライズドビューは、View Delta Join書き換えをサポートしていません。
- Hudi、Iceberg、またはJDBCカタログベースのマテリアライズドビューは、パーティションのインクリメンタルリフレッシュをサポートしていません。

## クエリ書き換えの設定

非同期マテリアライズドビュークエリの書き換えは、以下のセッション変数を通じて設定できます：

| **変数**                                       | **デフォルト** | **説明**                                              |
| ------------------------------------------- | ----------- | ------------------------------------------------------------ |
| enable_materialized_view_union_rewrite      | true        | マテリアライズドビューのユニオンクエリ書き換えを有効にするかどうかを制御するブール値。 |
| enable_rule_based_materialized_view_rewrite | true        | ルールベースのマテリアライズドビュークエリ書き換えを有効にするかどうかを制御するブール値。この変数は主に単一テーブルクエリの書き換えに使用されます。 |
| nested_mv_rewrite_max_level                 | 3           | クエリ書き換えに使用できるネストされたマテリアライズドビューの最大レベル。タイプ：INT。範囲：[1, +∞)。値 `1` は、他のマテリアライズドビュー上で作成されたマテリアライズドビューがクエリ書き換えに使用されないことを示します。 |

## クエリが書き換えられたかどうかを確認する

クエリが書き換えられたかどうかは、EXPLAINステートメントを使用してクエリプランを表示することで確認できます。`OlapScanNode`セクションの`TABLE`フィールドに対応するマテリアライズドビューの名前が表示されている場合、クエリがマテリアライズドビューに基づいて書き換えられたことを意味します。

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

## クエリ書き換えを無効にする

デフォルトでは、StarRocksはデフォルトカタログに基づいて作成された非同期マテリアライズドビューのクエリ書き換えを有効にしています。この機能を無効にするには、セッション変数 `enable_materialized_view_rewrite` を `false` に設定します。

外部カタログに基づいて作成された非同期マテリアライズドビューの場合、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用してマテリアライズドビューのプロパティ `force_external_table_query_rewrite` を `false` に設定することで、この機能を無効にできます。

## 制限事項

マテリアライズドビューに基づくクエリ書き換えについて、StarRocksには以下の制限があります：

- StarRocksは、rand、random、uuid、sleepなどの非決定的関数を含むクエリの書き換えをサポートしていません。
- StarRocksはウィンドウ関数を含むクエリの書き換えをサポートしていません。
- LIMIT、ORDER BY、UNION、EXCEPT、INTERSECT、MINUS、GROUPING SETS、WITH CUBE、またはWITH ROLLUPを含むステートメントで定義されたマテリアライズドビューは、クエリ書き換えに使用できません。
- ベーステーブルと外部カタログ上で構築されたマテリアライズドビュー間のクエリ結果の強い一貫性は保証されません。
- JDBCカタログのベーステーブル上で作成された非同期マテリアライズドビューはクエリ書き換えをサポートしていません。

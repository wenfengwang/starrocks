---
displayed_sidebar: "Japanese"
---

# マテリアライズドビューを使用したデータレイククエリの高速化

このトピックでは、StarRocksの非同期マテリアライズドビューを使用してデータレイク内のクエリのパフォーマンスを最適化する方法について説明します。

StarRocksは、データレイククエリ機能を提供しており、探索的なクエリやデータの分析に非常に効果的です。ほとんどのシナリオでは、[データキャッシュ](../data_source/data_cache.md)を使用することで、リモートストレージのジッターや大量のI / O操作によるパフォーマンスの低下を回避できます。

しかし、データレイクからの複雑で効率的なレポートの構築やこれらのクエリのさらなる高速化には、パフォーマンスの課題が残ることがあります。非同期マテリアライズドビューを使用すると、レポートやデータアプリケーションの高い並行性とクエリパフォーマンスを実現できます。

## 概要

StarRocksは、Hiveカタログ、Icebergカタログ、Hudiカタログなどの外部カタログを基にした非同期マテリアライズドビューの構築をサポートしています。外部カタログを基にしたマテリアライズドビューは、以下のシナリオで特に有用です。

- **データレイクレポートの透過的な高速化**

  データレイクレポートのクエリパフォーマンスを確保するために、データエンジニアは通常、データアナリストと密接に連携して、レポートの高速化レイヤーの構築ロジックを調査する必要があります。高速化レイヤーをさらに更新する必要がある場合は、構築ロジック、処理スケジュール、およびクエリステートメントを更新する必要があります。

  マテリアライズドビューのクエリリライト機能により、レポートの高速化をユーザーに見えないように透過的に行うことができます。遅いクエリが特定された場合、データエンジニアは遅いクエリのパターンを分析し、必要に応じてマテリアライズドビューを作成することができます。アプリケーション側のクエリは、マテリアライズドビューによってインテリジェントに書き換えられ、ビジネスアプリケーションのロジックやクエリステートメントを変更することなくクエリパフォーマンスが迅速に改善されます。

- **歴史データに関連するリアルタイムデータの増分計算**

  ビジネスアプリケーションが、StarRocksのネイティブテーブルにあるリアルタイムデータとデータレイクにある歴史データを関連付けて増分計算する必要がある場合。このような場合、マテリアライズドビューは簡単な解決策を提供できます。例えば、リアルタイムのファクトテーブルがStarRocksのネイティブテーブルで、次元テーブルがデータレイクに格納されている場合、ネイティブテーブルと外部データソースのテーブルを関連付けるマテリアライズドビューを構築することで、増分計算を簡単に行うことができます。

- **メトリックレイヤーの迅速な構築**

  メトリックの計算と処理は、データの高次元性を扱う際に課題が発生する場合があります。マテリアライズドビューを使用すると、データの事前集計やロールアップを行うことで、比較的軽量なメトリックレイヤーを作成することができます。さらに、マテリアライズドビューは自動的にリフレッシュされるため、メトリックの計算の複雑さがさらに軽減されます。

マテリアライズドビュー、データキャッシュ、StarRocksのネイティブテーブルは、すべてクエリパフォーマンスの大幅な向上を実現するための効果的な手段です。以下の表は、これらの主な違いを比較しています。

<table class="comparison">
  <thead>
    <tr>
      <th>&nbsp;</th>
      <th>データキャッシュ</th>
      <th>マテリアライズドビュー</th>
      <th>ネイティブテーブル</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>データの読み込みと更新</b></td>
      <td>クエリ実行時にデータキャッシュが自動的にトリガーされます。</td>
      <td>リフレッシュタスクが自動的にトリガーされます。</td>
      <td>さまざまなインポート方法をサポートしていますが、読み込みタスクの手動メンテナンスが必要です。</td>
    </tr>
    <tr>
      <td><b>データキャッシュの粒度</b></td>
      <td><ul><li>ブロックレベルのデータキャッシュをサポート</li><li>LRUキャッシュ削除メカニズムに従います</li><li>計算結果はキャッシュされません</li></ul></td>
      <td>計算済みのクエリ結果を格納します。</td>
      <td>テーブルスキーマに基づいてデータを格納します。</td>
    </tr>
    <tr>
      <td><b>クエリパフォーマンス</b></td>
      <td colspan="3" style={{textAlign: 'center'}} >データキャッシュ &le; マテリアライズドビュー = ネイティブテーブル</td>
    </tr>
    <tr>
      <td><b>クエリステートメント</b></td>
      <td><ul><li>データレイクに対するクエリステートメントの変更は必要ありません</li><li>クエリがキャッシュにヒットすると、計算が実行されます。</li></ul></td>
      <td><ul><li>データレイクに対するクエリステートメントの変更は必要ありません</li><li>クエリリライトを活用して計算済みの結果を再利用します</li></ul></td>
      <td>ネイティブテーブルをクエリするためにクエリステートメントの変更が必要です</td>
    </tr>
  </tbody>
</table>

<br />

データレイクから直接クエリするか、ネイティブテーブルにデータをロードする場合と比較して、マテリアライズドビューにはいくつかの独自の利点があります。

- **ローカルストレージの高速化**: マテリアライズドビューは、インデックス、パーティショニング、バケット、およびコロケートグループなど、StarRocksのローカルストレージの高速化の利点を活用することができます。これにより、データレイクからのデータクエリに比べてより良いクエリパフォーマンスが実現されます。
- **ロードタスクのゼロメンテナンス**: マテリアライズドビューは、自動リフレッシュタスクによってデータを透明に更新します。スケジュールされたデータの更新のためのロードタスクのメンテナンスは必要ありません。さらに、Hiveカタログベースのマテリアライズドビューは、データの変更を検出し、パーティションレベルでの増分リフレッシュを実行することができます。
- **インテリジェントなクエリリライト**: クエリは透過的にマテリアライズドビューを使用して書き換えることができます。アプリケーションで使用されるクエリステートメントを変更する必要なく、即座に高速化の恩恵を受けることができます。

<br />

そのため、以下のシナリオでマテリアライズドビューの使用をおすすめします。

- データキャッシュが有効になっている場合でも、クエリのパフォーマンスがクエリのレイテンシや並行性の要件に満たない場合。
- クエリに再利用可能なコンポーネント（固定の集計関数やジョインパターンなど）が含まれている場合。
- データがパーティションで構成され、クエリには比較的高いレベルでの集計が関与している場合（例：日別の集計）。

<br />

以下のシナリオでは、データキャッシュを介した高速化を優先することをおすすめします。

- クエリに再利用可能なコンポーネントが多く含まれており、データレイクから任意のデータをスキャンする可能性がある場合。
- リモートストレージには大きな変動や不安定性があり、アクセスに影響がある可能性がある場合。

## 外部カタログベースのマテリアライズドビューを作成する

外部カタログのテーブル上にマテリアライズドビューを作成する方法は、StarRocksのネイティブテーブル上のマテリアライズドビューを作成する方法と似ています。適切なリフレッシュ戦略を設定し、外部カタログベースのマテリアライズドビューのクエリリライトを手動で有効にする必要があります。

### 適切なリフレッシュ戦略を選択する

現在、StarRocksはHudiカタログおよびJDBCカタログでパーティションレベルのデータ変更を検出することができません。そのため、タスクがトリガーされるとフルサイズのリフレッシュが実行されます。

HiveカタログおよびIcebergカタログ（v3.1.4以降）では、StarRocksはパーティションレベルでのデータ変更を検出できます。これにより、StarRocksは次のことができます。

- データ変更があるパーティションのみをリフレッシュし、フルサイズのリフレッシュを回避することで、リフレッシュによるリソース消費を削減します。

- クエリリライト中に一定の程度のデータ整合性を確保します。データレイクのベーステーブルにデータの変更がある場合、クエリはマテリアライズドビューを使用するように書き換えられません。

  > **注意**
  >
  > マテリアライズドビューを作成する際にプロパティ`mv_rewrite_staleness_second`を設定することで、ある程度のデータ整合性を許容することもできます。詳細については、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

データをパーティションごとにリフレッシュする必要がある場合は、マテリアライズドビューのパーティショニングキーをベーステーブルのパーティショニングキーに含める必要があります。

Hiveカタログの場合、Hiveメタデータキャッシュリフレッシュ機能を有効にすることで、StarRocksがパーティションレベルでのデータ変更を検出できるようになります。この機能を有効にすると、StarRocksは定期的にHiveメタストアサービス（HMS）またはAWS Glueにアクセスして、最近クエリされたホットデータのメタデータ情報をチェックし、キャッシュされたメタデータを更新します。

Hiveメタデータキャッシュリフレッシュ機能を有効にするには、[ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)を使用して、次のFEダイナミック構成項目を設定できます。

| **構成項目**                                                  | **デフォルト**                  | **説明**                                                     |
| ------------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| enable_background_refresh_connector_metadata                 | true (v3.0) false (v2.5) | 定期的なHiveメタデータキャッシュリフレッシュを有効にするかどうか。有効にすると、StarRocksはHiveクラスタのメタストア（HiveメタストアまたはAWS Glue）をポーリングし、頻繁にアクセスされるHiveカタログのキャッシュされたメタデータをリフレッシュしてデータ変更を検出します。trueはHiveメタデータキャッシュリフレッシュを有効にすることを示し、falseは無効にすることを示します。 |
| background_refresh_metadata_interval_millis                  | 600000 (10分)              | 2つの連続するHiveメタデータキャッシュリフレッシュの間隔。単位：ミリ秒。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400 (24時間)           | Hiveカタログへのアクセスがある場合、アクセスしてから指定された時間より長い間アクセスされていない場合、StarRocksはキャッシュされたメタデータの更新を停止します。アクセスされていないHiveカタログの場合、StarRocksはキャッシュされたメタデータを更新しません。単位：秒。|

v3.1.4以降、StarRocksはIcebergカタログのパーティションレベルでのデータ変更を検出することができます。現在、Iceberg V1テーブルのみがサポートされています。

### 外部カタログベースのマテリアライズドビューのクエリリライトを有効にする
デフォルトでは、StarRocksはHudi、Iceberg、およびJDBCカタログ上で構築されたマテリアライズドビューのクエリのリライトをサポートしていません。なぜなら、このシナリオでのクエリのリライトは結果の強い一貫性を保証できないからです。マテリアライズドビューを作成する際に、プロパティ `force_external_table_query_rewrite` を `true` に設定してこの機能を有効にすることができます。Hiveカタログ上のテーブルで構築されたマテリアライズドビューでは、クエリのリライトがデフォルトで有効になっています。

例：

```SQL
CREATE MATERIALIZED VIEW ex_mv_par_tbl
PARTITION BY emp_date
DISTRIBUTED BY hash(empid)
PROPERTIES (
"force_external_table_query_rewrite" = "true"
) 
AS
select empid, deptno, emp_date
from `hive_catalog`.`emp_db`.`emps_par_tbl`
where empid < 5;
```

クエリのリライトを伴うシナリオでは、非常に複雑なクエリ文を使用してマテリアライズドビューを構築する場合は、クエリ文を分割して複数の単純なマテリアライズドビューをネストさせることをお勧めします。ネストされたマテリアライズドビューはより柔軟で、さまざまなクエリパターンに対応できます。

## ベストプラクティス

実際のビジネスシナリオでは、監査ログや[ビッグクエリログ](../administration/monitor_manage_big_queries.md#analyze-big-query-logs)を分析することで、実行遅延やリソース消費が高いクエリを特定できます。さらに、[クエリプロファイル](../administration/query_profile.md)を使用して、クエリの遅延が発生している具体的なステージを特定できます。以下のセクションでは、マテリアライズドビューを使用してデータレイクのクエリパフォーマンスを向上させる手順と例を提供します。

### ケース1: データレイクでの結合演算の高速化

マテリアライズドビューを使用して、データレイクでの結合クエリを高速化することができます。

次のように、Hiveカタログでの以下のクエリが特に遅いとします：

```SQL
--Q1
SELECT SUM(lo_extendedprice * lo_discount) AS REVENUE
FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates
WHERE
    lo_orderdate = d_datekey
    AND d_year = 1993
    AND lo_discount BETWEEN 1 AND 3
    AND lo_quantity < 25;

--Q2
SELECT SUM(lo_extendedprice * lo_discount) AS REVENUE
FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates
WHERE
    lo_orderdate = d_datekey
    AND d_yearmonth = 'Jan1994'
    AND lo_discount BETWEEN 4 AND 6
    AND lo_quantity BETWEEN 26 AND 35;

--Q3 
SELECT SUM(lo_revenue), d_year, p_brand
FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates, hive.ssb_1g_csv.part, hive.ssb_1g_csv.supplier
WHERE
    lo_orderdate = d_datekey
    AND lo_partkey = p_partkey
    AND lo_suppkey = s_suppkey
    AND p_brand BETWEEN 'MFGR#2221' AND 'MFGR#2228'
    AND s_region = 'ASIA'
GROUP BY d_year, p_brand
ORDER BY d_year, p_brand;
```

これらのクエリプロファイルを分析することで、クエリの実行時間の大部分が表 `lineorder` とその他の次元テーブルとのハッシュ結合に費やされていることがわかるかもしれません。

ここでは、Q1およびQ2は`lineorder`と`dates`を結合した後に集約を行い、Q3は`lineorder`、`dates`、`part`、および`supplier`を結合した後に集約を行います。

したがって、StarRocksの[View Delta Join rewrite](./query_rewrite_with_materialized_views.md#view-delta-join-rewrite)機能を活用して、`lineorder`、`dates`、`part`、および`supplier`を結合するマテリアライズドビューを構築することができます。

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES ( 
    -- ユニーク制約を指定します。
    "unique_constraints" = "
    hive.ssb_1g_csv.supplier.s_suppkey;
    hive.ssb_1g_csv.part.p_partkey;
    hive.ssb_1g_csv.dates.d_datekey",
    -- フォーリンキーを指定します。
    "foreign_key_constraints" = "
    hive.ssb_1g_csv.lineorder(lo_partkey) REFERENCES hive.ssb_1g_csv.part(p_partkey);
    hive.ssb_1g_csv.lineorder(lo_suppkey) REFERENCES hive.ssb_1g_csv.supplier(s_suppkey);
    hive.ssb_1g_csv.lineorder(lo_orderdate) REFERENCES hive.ssb_1g_csv.dates(d_datekey)",
    -- 外部カタログベースのマテリアライズドビューのクエリリライトを有効にします。
    "force_external_table_query_rewrite" = "TRUE"
)
AS SELECT
       l.LO_ORDERDATE AS LO_ORDERDATE,
       l.LO_ORDERKEY AS LO_ORDERKEY,
       l.LO_PARTKEY AS LO_PARTKEY,
       l.LO_SUPPKEY AS LO_SUPPKEY,
       l.LO_QUANTITY AS LO_QUANTITY,
       l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
       l.LO_DISCOUNT AS LO_DISCOUNT,
       l.LO_REVENUE AS LO_REVENUE,
       s.S_REGION AS S_REGION,
       p.P_BRAND AS P_BRAND,
       d.D_YEAR AS D_YEAR,
       d.D_YEARMONTH AS D_YEARMONTH
   FROM hive.ssb_1g_csv.lineorder AS l
            INNER JOIN hive.ssb_1g_csv.supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
            INNER JOIN hive.ssb_1g_csv.part AS p ON p.P_PARTKEY = l.LO_PARTKEY
            INNER JOIN hive.ssb_1g_csv.dates AS d ON l.LO_ORDERDATE = d.D_DATEKEY;
```

### ケース2: データレイクでの集約および結合による集計の高速化

マテリアライズドビューを使用して、単一のテーブルにおける集計クエリや複数のテーブルを結合した集計クエリを高速化することができます。

- 単一テーブルの集計クエリ

  単一のテーブルに関する典型的なクエリに対して、そのクエリプロファイルはAGGREGATEノードが多くの時間を消費していることを示します。一般的な集計演算子を使用して、マテリアライズドビューを構築することができます。

  以下のように、遅いクエリがあるとします：

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate limit 100;
  ```

  Q4は、ユニークな注文の日次件数を計算します。count distinctの計算は計算負荷が高い可能性があるため、以下の2種類のマテリアライズドビューを作成して高速化することができます：

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_1 
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  AS 
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  
  CREATE MATERIALIZED VIEW mv_2_2 
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  AS 
  SELECT
  -- lo_orderkeyはBIGINT型である必要があるため、クエリのリライトに使用できるようにする。 
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  この文脈では、リライトの失敗を避けるため、LIMITおよびORDER BY句を使用してマテリアライズドビューを作成しないように注意してください。クエリのリライトの制限に関する詳細は、[マテリアライズドビューを使用したクエリリライト - 制限事項](./query_rewrite_with_materialized_views.md#limitations)を参照してください。

- 複数テーブルの集計クエリ

  結合結果の集計を行うシナリオでは、既存のマテリアライズドビュー上にネストされたマテリアライズドビューを作成して、テーブルを結合してより多くの集計を行うことができます。たとえば、前述のケースの例に基づいて、Q1およびQ2を高速化するために、以下のようなマテリアライズドビューを作成することができます：

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_3
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  AS 
  SELECT
  lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth, SUM(lo_extendedprice * lo_discount) AS REVENUE
  FROM lineorder_flat_mv
  GROUP BY lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth;
  ```

  もちろん、結合および集計計算を単一のマテリアライズドビュー内で行うことも可能です。これらのタイプのマテリアライズドビューはクエリのリライトの機会が少ないかもしれませんが（特定の計算に起因するため）、通常は集計後のストレージスペースが少なくなります。選択は特定のユースケースに基づいて行うことができます。

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_4
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  PROPERTIES (
      "force_external_table_query_rewrite" = "TRUE"
  )
  AS
```SQL
SELECT lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth, SUM(lo_extendedprice * lo_discount) AS REVENUE
FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates
WHERE lo_orderdate = d_datekey
GROUP BY lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth;
```

### ケース3：データレイクでの結合の高速化

一部のシナリオでは、まず1つのテーブルで集計計算を行い、その後他のテーブルとの結合クエリを実行する必要があることがあります。StarRocksのクエリ書き換え機能をフルに活用するために、ネストされたマテリアライズドビューの構築をお勧めします。例：

```SQL
--Q5
SELECT * FROM  (
    SELECT 
      l.lo_orderkey, l.lo_orderdate, c.c_custkey, c_region, sum(l.lo_revenue)
    FROM 
      hive.ssb_1g_csv.lineorder l 
      INNER JOIN (
        SELECT distinct c_custkey, c_region 
        from 
          hive.ssb_1g_csv.customer 
        WHERE 
          c_region IN ('ASIA', 'AMERICA') 
      ) c ON l.lo_custkey = c.c_custkey
      GROUP BY  l.lo_orderkey, l.lo_orderdate, c.c_custkey, c_region
  ) c1 
WHERE 
  lo_orderdate = '19970503';
```

Q5は、最初に`customer`テーブルで集計クエリを実行し、その後`lineorder`テーブルとの結合および集計を行います。類似のクエリでは、`c_region`および`lo_orderdate`に異なるフィルタを適用することがあります。クエリ書き換え機能を活用するために、集計と結合のために2つのマテリアライズドビューを作成できます。

```SQL
--mv_3_1
CREATE MATERIALIZED VIEW mv_3_1
DISTRIBUTED BY HASH(c_custkey)
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES (
    "force_external_table_query_rewrite" = "TRUE"
)
AS
SELECT distinct c_custkey, c_region from hive.ssb_1g_csv.customer; 

--mv_3_2
CREATE MATERIALIZED VIEW mv_3_2
DISTRIBUTED BY HASH(lo_orderdate)
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES (
    "force_external_table_query_rewrite" = "TRUE"
)
AS
SELECT l.lo_orderdate, l.lo_orderkey, mv.c_custkey, mv.c_region, sum(l.lo_revenue)
FROM hive.ssb_1g_csv.lineorder l 
INNER JOIN mv_3_1 mv
ON l.lo_custkey = mv.c_custkey
GROUP BY l.lo_orderkey, l.lo_orderdate, mv.c_custkey, mv.c_region;
```

### ケース4：データレイクでのリアルタイムデータと過去データのホットデータとコールドデータの分離

次のシナリオを考えてみましょう：直近3日間に更新されたデータは直接StarRocksに書き込まれ、それ以前のデータは確認され、Hiveにバッチ書き込みされます。ただし、過去7日間のデータを含むクエリはまだあります。この場合、データを自動的に期限切れにするための簡単なモデルとマテリアライズドビューを作成できます。

```SQL
CREATE MATERIALIZED VIEW mv_4_1 
DISTRIBUTED BY HASH(lo_orderdate)
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
AS 
SELECT lo_orderkey, lo_orderdate, lo_revenue
FROM hive.ssb_1g_csv.lineorder
WHERE lo_orderdate<=current_date()
AND lo_orderdate>=date_add(current_date(), INTERVAL -4 DAY);
```

上位レイヤーアプリケーションのロジックに基づいてそれに基づいてさらなるビューまたはマテリアライズドビューを構築することができます。
```
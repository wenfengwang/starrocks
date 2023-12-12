---
displayed_sidebar: "Japanese"
---

# マテリアライズドビューを使用したデータレイクのクエリアクセラレーション

このトピックでは、StarRocksの非同期マテリアライズドビューを使用してデータレイク内のクエリパフォーマンスを最適化する方法について説明します。

StarRocksでは、データレイクのクエリ機能を提供しており、エクスプローラリクエリやデータの解析に非常に効果的です。ほとんどのシナリオでは、[データキャッシュ](../data_source_ja/data_cache_ja.md)を使用してブロックレベルのファイルキャッシングを提供し、リモートストレージの揺らぎや大量のI/O操作によるパフォーマンスの低下を回避することができます。

しかし、データレイクからのデータを使用した複雑で効率的なレポートの作成やこれらのクエリのさらなる高速化を行う場合、パフォーマンスの課題に直面することがあります。非同期マテリアライズドビューを使用すると、レポートやデータアプリケーションのクエリパフォーマンスを向上させるためのより高い同時実行性と優れたクエリパフォーマンスを実現できます。

## 概要

StarRocksでは、Hiveカタログ、Icebergカタログ、Hudiカタログなどの外部カタログを基にした非同期マテリアライズドビューの構築をサポートしています。外部カタログベースのマテリアライズドビューは、次のようなシナリオで特に有効です。

- **データレイクレポートの透過的なアクセラレーション**

  データレイクレポートのクエリパフォーマンスを確保するために、データエンジニアは通常、データアナリストと緊密に連携してレポートのアクセラレーションレイヤの構築ロジックを探究する必要があります。アクセラレーションレイヤがさらなる更新を必要とする場合、構築ロジック、処理スケジュール、クエリステートメントを更新する必要があります。

  マテリアライズドビューのクエリ書き換え機能により、レポートのアクセラレーションをユーザーには透過的かつ感知できないようにすることができます。遅いクエリが特定された場合、データエンジニアは遅いクエリのパターンを分析し、必要に応じてマテリアライズドビューを作成することができます。アプリケーション側のクエリはインテリジェントに書き換えられ、マテリアライズドビューによって透過的にアクセラレーションされるため、ビジネスアプリケーションやクエリステートメントのロジックを変更することなく、クエリパフォーマンスの迅速な改善が可能です。

- **履歴データに関連するリアルタイムデータの増分計算**

  ビジネスアプリケーションには、StarRocksネイティブテーブルのリアルタイムデータとデータレイクの履歴データを関連付けて増分計算する必要がある場合があります。このような場合、マテリアライズドビューを使用することで簡単に解決策を提供できます。たとえば、リアルタイムファクトテーブルがStarRocksのネイティブテーブルであり、ディメンションテーブルがデータレイクに格納されている場合、外部データソースのテーブルとネイティブテーブルを関連付けるマテリアライズドビューを構築することで、増分計算を簡単に行うことができます。

- **メトリクスレイヤの迅速な構築**

  メトリクスの計算と処理は、データの高次元性を処理する際に課題に直面する場合があります。マテリアライズドビューを使用することで、データの事前集計とロールアップを行うことができるため、比較的軽量なメトリクスレイヤを作成することができます。さらに、マテリアライズドビューは自動的にリフレッシュされるため、メトリクス計算の複雑さがさらに低減されます。

マテリアライズドビュー、データキャッシュ、およびStarRocksのネイティブテーブルは、クエリパフォーマンスの劇的な向上を実現するための効果的な手段です。以下の表は、それらの主な違いを比較したものです：

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
      <td><b>データロードと更新</b></td>
      <td>クエリによってデータキャッシュが自動的にトリガされます。</td>
      <td>リフレッシュタスクは自動的にトリガされます。</td>
      <td>さまざまなインポートメソッドをサポートしますが、インポートタスクの手動メンテナンスが必要です。</td>
    </tr>
    <tr>
      <td><b>データキャッシュの粒度</b></td>
      <td><ul><li>ブロックレベルのデータキャッシュをサポート</li><li>LRUキャッシュ削除メカニズムに従う</li><li>計算結果はキャッシュされない</li></ul></td>
      <td>事前計算されたクエリ結果を保存</td>
      <td>テーブルスキーマに基づいてデータを保存</td>
    </tr>
    <tr>
      <td><b>クエリパフォーマンス</b></td>
      <td colspan="3" style={{textAlign: 'center'}}>データキャッシュ &le; マテリアライズドビュー = ネイティブテーブル</td>
    </tr>
    <tr>
      <td><b>クエリステートメント</b></td>
      <td><ul><li>データレイクへのクエリステートメントを変更する必要はありません</li><li>クエリがキャッシュにヒットすると、計算が行われます。</li></ul></td>
      <td><ul><li>データレイクへのクエリステートメントを変更する必要はありません</li><li>クエリ書き換えを利用して事前計算結果を再利用</li></ul></td>
      <td>ネイティブテーブルをクエリするためにクエリステートメントの変更が必要です</td>
    </tr>
  </tbody>
</table>

<br />

データレイクのデータを直接クエリしたり、データをネイティブテーブルにロードするよりも、マテリアライズドビューにはいくつかの特徴的な利点があります：

- **ローカルストレージでのアクセラレーション**：マテリアライズドビューは、インデックス、パーティション分割、バケット分散、およびコロケートグループなど、StarRocksのローカルストレージによるアクセラレーションの利点を活用できるため、データレイクからのデータクエリよりも優れたクエリパフォーマンスが実現できます。
- **ロードタスクのゼロメンテナンス**：マテリアライズドビューは、自動リフレッシュタスクを介してデータを透過的に更新します。スケジュールされたデータ更新を行うためのロードタスクのメンテナンスは不要です。また、Hiveカタログベースのマテリアライズドビューは、データの変更を検知してパーティションレベルでの増分リフレッシュを実行することができます。
- **インテリジェントなクエリ書き換え**：クエリはマテリアライズドビューを使用して透過的に書き換えられます。アプリケーションで使用しているクエリステートメントを変更する必要なく、すぐにアクセラレーションの恩恵を受けることができます。

<br />

したがって、次のシナリオでは、マテリアライズドビューの使用をお勧めします：

- データキャッシュが有効になっているにもかかわらず、クエリパフォーマンスがクエリレイテンシや同時実行性の要件を満たしていない場合。
- クエリに再利用可能なコンポーネントが含まれており、固定した集計関数や結合パターンが存在する場合。
- データがパーティションで整理されており、クエリに比較的高いレベルの集計（日ごとの集計など）が含まれる場合。

<br />

以下のシナリオでは、データキャッシュを通じたアクセラレーションを優先することをお勧めします：

- クエリに再利用可能なコンポーネントが含まれておらず、データレイクから任意のデータをスキャンする可能性がある場合。
- リモートストレージには大きな変動や不安定性があり、アクセスに潜在的な影響がある場合。

## 外部カタログベースのマテリアライズドビューの作成

外部カタログのテーブル上にマテリアライズドビューを作成する方法は、StarRiocksのネイティブテーブル上のマテリアライズドビューを作成する方法と似ています。適切なリフレッシュ戦略を設定し、外部カタログベースのマテリアライズドビューに対してクエリ書き換えを手動で有効にする必要があります。

### 適切なリフレッシュ戦略を選択する

現在、StarRocksは、HudiカタログやJDBCカタログのパーティションレベルのデータ変更を検出することができません。したがって、タスクがトリガされると、フルサイズのリフレッシュが実行されます。

HiveカタログやIcebergカタログ（v3.1.4以降）では、StarRocksはパーティションレベルでのデータ変更を検出できます。その結果、StarRocksでは次のことが可能です。

- データの変更があるパーティションのみをリフレッシュし、フルサイズのリフレッシュを回避することで、リフレッシュによるリソース消費を低減します。

- リフレッシュ時のデータの整合性をある程度確保します。データレイクの基本テーブルにデータの変更がある場合、クエリはマテリアライズドビューを使用して書き換えられることはありません。

  > **注意**
  >
  > マテリアライズドビューを作成する際にプロパティ`mv_rewrite_staleness_second`を設定することで、一定レベルのデータ不整合を許容することも可能です。詳細については、[CREATE MATERIALIZED VIEW](../sql-reference_ja/sql-statements_ja/data-definition_ja/CREATE_MATERIALIZED_VIEW_ja.md)を参照してください。

なお、パーティションごとにリフレッシュする必要がある場合、マテリアライズドビューのパーティショニングキーは、基になるテーブルのパーティショニングキーに含まれている必要があります。

Hiveカタログでは、Hiveメタデータキャッシュのリフレッシュ機能を有効にすることで、パーティションレベルでのデータ変更をStarRocksが検出できるようにすることができます。この機能を有効にすると、StarRocksは定期的にHiveメタストアサービス（HiveメタストアまたはAWS Glue）にアクセスし、最近クエリされたホットデータのメタデータ情報を確認してキャッシュされたメタデータをリフレッシュします。

Hiveメタデータキャッシュのリフレッシュ機能を有効にするには、[ADMIN SET FRONTEND CONFIG](../sql-reference_ja/sql-statements_ja/Administration_ja/ADMIN_SET_CONFIG_ja.md)を使用して次のFE動的構成項目を設定できます：

| **設定項目**                                                  | **デフォルト値**           | **説明**                                                     |
| ------------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| enable_background_refresh_connector_metadata                 | v3.0ではtrue、v2.5ではfalse | 定期的なHiveメタデータキャッシュのリフレッシュを有効にするかどうか。有効にすると、StarRocksはHiveクラスタのメタストア（HiveメタストアまたはAWS Glue）をポーリングして、頻繁にアクセスされるHiveカタログのキャッシュメタデータをリフレッシュし、データの変更を感知します。trueはHiveメタデータキャッシュリフレッシュを有効にし、falseは無効にします。 |
| background_refresh_metadata_interval_millis                  | 600000 (10分)              | 2回の連続するHiveメタデータキャッシュリフレッシュ間のインターバル。ミリ秒単位。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400 (24時間)             | Hiveカタログにアクセスされたままになっている場合、指定した時間以上アクセスされていない場合、StarRocksはキャッシュされたメタデータのリフレッシュを停止します。アクセスされていない場合、StarRocksはキャッシュされたメタデータをリフレッシュしません。秒単位。 |

v3.1.4以降、StarRocksはIcebergカタログのパーティションレベルのデータ変更を検出することができます。現在、Iceberg V1テーブルのみがサポートされています。

### 外部カタログベースのマテリアライズドビューのクエリ書き換えを有効にする
デフォルトでは、StarRocksはHudi、Iceberg、JDBCカタログで構築されたマテリアライズドビューに対するクエリの書き換えをサポートしていません。なぜなら、このシナリオでのクエリの書き換えは結果の強力な一貫性を確保できないためです。マテリアライズドビューを作成する際に、`force_external_table_query_rewrite`プロパティを`true`に設定することで、この機能を有効にすることができます。Hiveカタログのテーブルで構築されたマテリアライズドビューでは、クエリの書き換えがデフォルトで有効になっています。

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

クエリの書き換えを伴うシナリオでは、非常に複雑なクエリ文を使用してマテリアライズドビューを構築する場合、クエリ文を分割し、階層的に複数の単純なマテリアライズドビューを構築することをお勧めします。階層的なマテリアライズドビューはより柔軟で、より広範囲のクエリパターンに対応できます。

## ベストプラクティス

実際のビジネスシナリオでは、監査ログや[big query logs](../administration/monitor_manage_big_queries.md#analyze-big-query-logs)を分析して、実行レイテンシが高いクエリやリソース消費が多いクエリを特定することができます。さらに、[query profiles](../administration/query_profile.md)を使用して、クエリの遅延が発生している具体的なステージを特定することもできます。以下のセクションでは、マテリアライズドビューを使用してデータレイククエリのパフォーマンスを向上させる方法についての手順と例を提供します。

### ケース1: データレイクでの結合計算の高速化

データレイクでの結合クエリを高速化するためにマテリアライズドビューを使用することができます。

次のように、Hiveカタログ上の以下のクエリが特に遅い場合を考えてみましょう：

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

それらのクエリプロファイルを分析することで、クエリの実行時間の大部分がテーブル`lineorder`と他のディメンションテーブルの`lo_orderdate`列を基準としたハッシュ結合に費やされていることに気付くかもしれません。

ここで、Q1とQ2は`lineorder`と`dates`を結合した後に集計を行い、Q3は`lineorder`、`dates`、`part`、`supplier`を結合した後に集計を行っています。

したがって、StarRocksの[View Delta Join rewrite](./query_rewrite_with_materialized_views.md#view-delta-join-rewrite)機能を利用して、`lineorder`、`dates`、`part`、`supplier`を結合するマテリアライズドビューを構築することができます。

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES ( 
    -- ユニークな制約を指定します。
    "unique_constraints" = "
    hive.ssb_1g_csv.supplier.s_suppkey;
    hive.ssb_1g_csv.part.p_partkey;
    hive.ssb_1g_csv.dates.d_datekey",
    -- 外部カタログベースのマテリアライズドビューのクエリの書き換えを有効にします。
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

### ケース2: データレイクでの集計および結合による集計の高速化

マテリアライズドビューを使用して、単一テーブルや複数のテーブルを使った集計クエリを高速化することが可能です。

- 単一テーブルの集計クエリ

  単一テーブルの典型的なクエリに対して、そのクエリプロファイルを見るとAGGREGATEノードの時間がかなりかかることがわかるでしょう。共通の集計演算子を使用してマテリアライズドビューを構築することができます。

  たとえば、次のような遅いクエリがあるとします：

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate limit 100;
  ```

  Q4は一意な注文数の日次集計を行っています。一意な数の計算が計算上コストがかかるため、次の2つのタイプのマテリアライズドビューを作成して高速化することができます：

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
  -- lo_orderkey must be the BIGINT type so that it can be used for query rewrite.
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  この場合、リライトの失敗を避けるために、LIMITやORDER BY句を使用してマテリアライズドビューを作成しないでください。クエリの書き換えの制限についての詳細情報については、[Query rewrite with materialized views - Limitations](./query_rewrite_with_materialized_views.md#limitations)を参照してください。

- 複数のテーブルでの集計クエリ

  結果の集計を含むシナリオでは、既存のマテリアライズドビュー上にネストされたマテリアライズドビューを作成して、テーブルを結合してから結果をさらに集計することができます。たとえば、ケース1の例に基づいて、Q1とQ2の集計パターンが似ているため、次のようなマテリアライズドビューを作成することができます：

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

  もちろん、単一のマテリアライズドビューで結合および集計計算を両方行うことも可能です。これらのタイプのマテリアライズドビューは、特定の計算によるリライトの機会が少ないですが、集計後に通常よりも少ないストレージを占有します。それぞれの選択は特定のユースケースに基づいて行うことができます。

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

### ケース3：データレイクにおける集計の結合の加速

一部のシナリオでは、最初に1つのテーブルで集計計算を行い、その後他のテーブルとの結合クエリを実行する必要がある場合があります。StarRocksのクエリ書き換え機能を十分に活用するためには、ネストされたマテリアライズドビューを構築することをお勧めします。例えば：

```SQL
--Q5
SELECT * FROM (
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
      GROUP BY l.lo_orderkey, l.lo_orderdate, c.c_custkey, c_region
  ) c1 
WHERE 
  lo_orderdate = '19970503';
```

Q5は最初に`customer`テーブルで集計クエリを実行し、その後`lineorder`テーブルとの結合および集計を行います。類似するクエリでは`c_region`および`lo_orderdate`の異なるフィルタを含む場合があります。クエリ書き換え機能を活用するためには、集計用のマテリアライズドビューと結合用のもう1つのマテリアライズドビューを作成できます。

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

### ケース4：データレイクにおけるリアルタイムデータと履歴データのホットデータとコールドデータの分離

以下のシナリオを考慮してください：過去3日間内に更新されたデータは直接StarRocksに書き込まれる一方、それより新しいデータは確認され、Hiveにバッチ書き込みされます。しかし、過去7日間のデータが関与するクエリも存在します。この場合、データを自動的に期限切れにするために単純なモデルを作成し、マテリアライズドビューを使用することができます。

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

上位のアプリケーションのロジックに基づいて、さらにその上にビューやマテリアライズドビューを構築することができます。
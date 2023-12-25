---
displayed_sidebar: English
---

# データレイククエリの高速化のためのマテリアライズドビュー

このトピックでは、StarRocksの非同期マテリアライズドビューを使用してデータレイクのクエリパフォーマンスを最適化する方法について説明します。

StarRocksは、箱から出してすぐに使用できるデータレイククエリ機能を提供し、レイク内のデータの探索的クエリや分析に非常に効果的です。多くのシナリオでは、[Data Cache](../data_source/data_cache.md)はブロックレベルのファイルキャッシングを提供し、リモートストレージのジッターや多数のI/O操作によるパフォーマンスの劣化を避けることができます。

しかし、レイクからのデータを使用して複雑で効率的なレポートを構築したり、これらのクエリをさらに加速したりする場合、パフォーマンスの課題に直面することがあります。非同期マテリアライズドビューを使用することで、レイク上のレポートやデータアプリケーションの同時実行性とクエリパフォーマンスを向上させることができます。

## 概要

StarRocksは、Hiveカタログ、Icebergカタログ、Hudiカタログなどの外部カタログに基づいた非同期マテリアライズドビューの構築をサポートしています。外部カタログに基づいたマテリアライズドビューは、特に以下のシナリオで有用です：

- **データレイクレポートの透過的な加速**

  データレイクレポートのクエリパフォーマンスを確保するため、データエンジニアは通常、データアナリストと密接に連携して、レポートの加速層の構築ロジックを探る必要があります。加速層にさらなる更新が必要な場合、構築ロジック、処理スケジュール、およびクエリステートメントをそれに応じて更新する必要があります。

  マテリアライズドビューのクエリ書き換え機能を通じて、レポートの加速をユーザーに気づかれずに透過的に行うことができます。遅いクエリが特定された場合、データエンジニアは遅いクエリのパターンを分析し、必要に応じてマテリアライズドビューを作成できます。その後、アプリケーション側のクエリはマテリアライズドビューによって賢く書き換えられ、透過的に加速されるため、ビジネスアプリケーションのロジックやクエリステートメントを変更することなく、クエリパフォーマンスを迅速に向上させることができます。

- **リアルタイムデータと履歴データの関連付けによる増分計算**

  あなたのビジネスアプリケーションが、StarRocksのネイティブテーブルのリアルタイムデータとデータレイクの履歴データを関連付けて増分計算を行うことを要求する場合、マテリアライズドビューは簡単な解決策を提供します。例えば、リアルタイムのファクトテーブルがStarRocksのネイティブテーブルで、ディメンションテーブルがデータレイクに格納されている場合、ネイティブテーブルと外部データソースのテーブルを関連付けるマテリアライズドビューを構築することで、簡単に増分計算を実行できます。

- **メトリックレイヤーの迅速な構築**

  データの高次元性に対処する際に、メトリックの計算と処理は課題に直面することがあります。データの事前集約とロールアップを行うマテリアライズドビューを使用することで、比較的軽量なメトリックレイヤーを作成できます。さらに、マテリアライズドビューは自動的にリフレッシュされるため、メトリック計算の複雑さをさらに低減します。

StarRocksのマテリアライズドビュー、データキャッシュ、およびネイティブテーブルは、クエリパフォーマンスを大幅に向上させる効果的な手段です。以下の表は、それらの主な違いを比較したものです：

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
      <td>クエリによって自動的にデータキャッシュがトリガされます。</td>
      <td>リフレッシュタスクは自動的にトリガされます。</td>
      <td>様々なインポート方法をサポートしていますが、インポートタスクの手動メンテナンスが必要です。</td>
    </tr>
    <tr>
      <td><b>データキャッシングの粒度</b></td>
      <td><ul><li>ブロックレベルのデータキャッシングをサポート</li><li>LRUキャッシュ排出メカニズムに従う</li><li>計算結果はキャッシュされません</li></ul></td>
      <td>事前計算されたクエリ結果を格納します</td>
      <td>テーブルスキーマに基づいてデータを格納します</td>
    </tr>
    <tr>
      <td><b>クエリパフォーマンス</b></td>
      <td colspan="3" style={{textAlign: 'center'}}>データキャッシュ &le; マテリアライズドビュー = ネイティブテーブル</td>
    </tr>
    <tr>
      <td><b>クエリステートメント</b></td>
      <td><ul><li>データレイクに対するクエリステートメントを変更する必要はありません</li><li>クエリがキャッシュにヒットすると、計算が行われます。</li></ul></td>
      <td><ul><li>データレイクに対するクエリステートメントを変更する必要はありません</li><li>事前計算された結果を再利用するためにクエリ書き換えを活用します</li></ul></td>
      <td>ネイティブテーブルをクエリするためにクエリステートメントを変更する必要があります</td>
    </tr>
  </tbody>
</table>

<br />

レイクデータを直接クエリするか、ネイティブテーブルにデータをロードするのと比較して、マテリアライズドビューはいくつかのユニークな利点を提供します：

- **ローカルストレージの加速**: マテリアライズドビューは、インデックス、パーティショニング、バケッティング、コロケートグループなど、ローカルストレージのStarRocksの加速機能を活用できるため、データレイクから直接クエリする場合と比較して、より良いクエリパフォーマンスを提供します。
- **ローディングタスクのメンテナンス不要**: マテリアライズドビューは、自動リフレッシュタスクを通じてデータを透過的に更新します。スケジュールされたデータ更新を実行するためのローディングタスクのメンテナンスは不要です。さらに、Hiveカタログベースのマテリアライズドビューは、データの変更を検出し、パーティションレベルでの増分リフレッシュを実行することができます。
- **インテリジェントなクエリ書き換え**: クエリはマテリアライズドビューを使用するように透過的に書き換えられます。アプリケーションで使用するクエリステートメントを変更することなく、即座に加速のメリットを享受できます。

<br />

したがって、次のようなシナリオではマテリアライズドビューの使用を推奨します：

- データキャッシュが有効にもかかわらず、クエリのパフォーマンスがクエリのレイテンシと同時実行性の要件を満たしていない場合。

- クエリには、固定集計関数や結合パターンなどの再利用可能なコンポーネントが含まれています。
- データはパーティションに整理され、クエリでは比較的高いレベルでの集計が行われます（例：日別の集計）。

<br />

次のシナリオでは、データキャッシュを通じた高速化を優先することを推奨します：

- クエリには多くの再利用可能なコンポーネントが含まれておらず、データレイクから任意のデータをスキャンする可能性があります。
- リモートストレージの変動が大きいか不安定で、アクセスに潜在的な影響がある場合。

## 外部カタログに基づくマテリアライズドビューの作成

外部カタログのテーブルにマテリアライズドビューを作成することは、StarRocksのネイティブテーブルにマテリアライズドビューを作成することと似ています。必要なのは、使用しているデータソースに応じた適切なリフレッシュ戦略を設定し、外部カタログベースのマテリアライズドビューのクエリ書き換えを手動で有効にすることです。

### 適切なリフレッシュ戦略を選択

現在、StarRocksはHudiカタログやJDBCカタログでのパーティションレベルのデータ変更を検出することはできません。そのため、タスクがトリガーされたときにはフルサイズのリフレッシュが実行されます。

HiveカタログとIcebergカタログ（v3.1.4から）では、StarRocksはパーティションレベルでのデータ変更を検出することをサポートしています。その結果、StarRocksは以下を実現できます：

- データ変更があったパーティションのみをリフレッシュし、フルサイズのリフレッシュを避けてリソース消費を削減します。

- クエリ書き換え時にある程度のデータ整合性を保証します。データレイクのベーステーブルにデータ変更がある場合、クエリはマテリアライズドビューを使用するように書き換えられません。

  > **注記**
  >
  > マテリアライズドビューの作成時に`mv_rewrite_staleness_second`プロパティを設定することで、ある程度のデータ不整合を許容することもできます。詳細については、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

パーティションごとにリフレッシュする必要がある場合、マテリアライズドビューのパーティションキーはベーステーブルのそれに含まれている必要があります。

Hiveカタログについては、Hiveメタデータキャッシュのリフレッシュ機能を有効にすることで、StarRocksがパーティションレベルでのデータ変更を検出できるようになります。この機能が有効になると、StarRocksは定期的にHive Metastore Service（HMS）またはAWS Glueにアクセスして、最近クエリされたホットデータのメタデータ情報を確認します。

Hiveメタデータキャッシュのリフレッシュ機能を有効にするには、[ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)を使用して以下のFE動的設定項目を設定できます：

| **設定項目**                                                | **デフォルト**                | **説明**                                                      |
| ------------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| enable_background_refresh_connector_metadata                 | v3.0ではtrue、v2.5ではfalse | Hiveメタデータキャッシュの定期的なリフレッシュを有効にするかどうか。有効にすると、StarRocksはHiveクラスターのメタストア（Hive MetastoreまたはAWS Glue）をポーリングし、頻繁にアクセスされるHiveカタログのキャッシュされたメタデータを更新してデータ変更を検出します。trueはリフレッシュを有効にすることを示し、falseは無効にすることを示します。 |
| background_refresh_metadata_interval_millis                  | 600000（10分）             | 2回連続するHiveメタデータキャッシュのリフレッシュ間の間隔。単位はミリ秒です。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400（24時間）            | Hiveメタデータキャッシュのリフレッシュタスクの有効期限。アクセスされたHiveカタログについて、指定された時間以上アクセスされていない場合、StarRocksはそのキャッシュされたメタデータのリフレッシュを停止します。アクセスされていないHiveカタログについては、StarRocksはそのキャッシュされたメタデータをリフレッシュしません。単位は秒です。 |

v3.1.4から、StarRocksはIcebergカタログのパーティションレベルでのデータ変更を検出することをサポートしています。現在、Iceberg V1テーブルのみがサポートされています。

### 外部カタログベースのマテリアライズドビューのクエリ書き換えを有効にする

デフォルトでは、StarRocksはHudi、Iceberg、JDBCカタログに構築されたマテリアライズドビューのクエリ書き換えをサポートしていません。この機能を有効にするには、マテリアライズドビューを作成する際に`force_external_table_query_rewrite`プロパティを`true`に設定します。Hiveカタログのテーブルに構築されたマテリアライズドビューについては、クエリ書き換えがデフォルトで有効です。

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

クエリ書き換えを伴うシナリオでは、非常に複雑なクエリステートメントを使用してマテリアライズドビューを構築する場合、クエリステートメントを分割して、入れ子になった方法で複数のシンプルなマテリアライズドビューを構築することを推奨します。入れ子になったマテリアライズドビューはより汎用性が高く、より広範なクエリパターンに適応できます。

## ベストプラクティス

実際のビジネスシナリオでは、監査ログや[ビッグクエリログ](../administration/monitor_manage_big_queries.md#analyze-big-query-logs)を分析することで、実行待ち時間やリソース消費が多いクエリを特定できます。さらに、[クエリプロファイル](../administration/query_profile.md)を使用して、クエリが遅い特定のステージを特定することができます。以下のセクションでは、マテリアライズドビューを使用してデータレイククエリのパフォーマンスを向上させる方法についての指示と例を提供します。

### ケースワン：データレイクでの結合計算を加速

マテリアライズドビューを使用して、データレイクでの結合クエリを加速できます。

例えば、Hiveカタログに対する以下のクエリが特に遅いとします：

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

クエリプロファイルを分析すると、クエリの実行時間の大部分が、`lineorder`テーブルと他のディメンションテーブルとの間で`lo_orderdate`列に基づいたハッシュ結合に費やされていることがわかります。

ここでは、Q1とQ2は`lineorder`と`dates`を結合した後に集約を行い、Q3は`lineorder`、`dates`、`part`、`supplier`を結合した後に集約を行います。

したがって、StarRocksの[View Delta Join rewrite](./query_rewrite_with_materialized_views.md#view-delta-join-rewrite)機能を利用して、`lineorder`、`dates`、`part`、`supplier`を結合するマテリアライズドビューを構築できます。

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
    -- 外部キーを指定します。
    "foreign_key_constraints" = "
    hive.ssb_1g_csv.lineorder(lo_partkey) REFERENCES hive.ssb_1g_csv.part(p_partkey);
    hive.ssb_1g_csv.lineorder(lo_suppkey) REFERENCES hive.ssb_1g_csv.supplier(s_suppkey);
    hive.ssb_1g_csv.lineorder(lo_orderdate) REFERENCES hive.ssb_1g_csv.dates(d_datekey)",
    -- 外部カタログベースのマテリアライズドビューに対してクエリ書き換えを有効にします。
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

### ケース2: データレイクでの集計と結合の集計を高速化する

マテリアライズドビューは、単一テーブルの集計クエリでも複数テーブルを含む集計クエリでも、集計クエリを高速化するために使用できます。

- 単一テーブルの集計クエリ

  単一テーブルに対する典型的なクエリでは、そのクエリプロファイルがAGGREGATEノードが大量の時間を消費していることを示しています。一般的な集計演算子を使用してマテリアライズドビューを構築できます。

  以下が低速なクエリであると仮定します：

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate LIMIT 100;
  ```

  Q4は、日別のユニークな注文数を計算します。count distinctの計算は計算コストが高いため、以下の2種類のマテリアライズドビューを作成して高速化できます：

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
  -- lo_orderkeyはBIGINT型である必要があります。これにより、クエリ書き換えに使用できます。
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  このコンテキストでは、書き換えの失敗を避けるために、LIMIT句とORDER BY句を使用してマテリアライズドビューを作成しないでください。クエリ書き換えの制限についての詳細は、[マテリアライズドビューを使用したクエリ書き換え - 制限](./query_rewrite_with_materialized_views.md#limitations)を参照してください。

- 複数テーブルの集計クエリ

  結合結果の集計を含むシナリオでは、テーブルを結合する既存のマテリアライズドビューにネストされたマテリアライズドビューを作成して、結合結果をさらに集計できます。例えば、ケース1の例に基づいて、Q1とQ2の集計パターンが似ているため、以下のマテリアライズドビューを作成して高速化できます：

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

  もちろん、結合と集計の計算を1つのマテリアライズドビュー内で行うことも可能です。これらのタイプのマテリアライズドビューは、特定の計算のためにクエリ書き換えの機会が少ないかもしれませんが、通常は集計後により少ないストレージスペースを占有します。選択は、特定のユースケースに基づいて行うことができます。

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_4
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  PROPERTIES (
      "force_external_table_query_rewrite" = "TRUE"
  )
  AS
  SELECT lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth, SUM(lo_extendedprice * lo_discount) AS REVENUE
  FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates
  WHERE lo_orderdate = d_datekey
  GROUP BY lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth;
  ```

### ケース3: データレイクでの集計上の結合を高速化する

一部のシナリオでは、まず1つのテーブルで集計計算を行い、その後他のテーブルとの結合クエリを実行する必要があります。StarRocksのクエリ書き換え機能を最大限に活用するためには、ネストされたマテリアライズドビューを構築することを推奨します。例えば：

```SQL
--Q5
SELECT * FROM (
    SELECT 
      l.lo_orderkey, l.lo_orderdate, c.c_custkey, c_region, SUM(l.lo_revenue)
    FROM 
      hive.ssb_1g_csv.lineorder l 
      INNER JOIN (
        SELECT DISTINCT c_custkey, c_region 
        FROM 
          hive.ssb_1g_csv.customer 
        WHERE 
          c_region IN ('ASIA', 'AMERICA') 
      ) c ON l.lo_custkey = c.c_custkey
    GROUP BY l.lo_orderkey, l.lo_orderdate, c.c_custkey, c_region
  ) c1 
WHERE 
  lo_orderdate = '19970503';
```

Q5は、まず`customer`テーブルで集計クエリを実行し、次に`lineorder`テーブルとの結合と集計を行います。類似のクエリには、`c_region`や`lo_orderdate`に対する異なるフィルターが含まれる場合があります。クエリ書き換え機能を活用するためには、集計用と結合用の2つのマテリアライズドビューを作成できます。

```SQL
--mv_3_1
CREATE MATERIALIZED VIEW mv_3_1
DISTRIBUTED BY HASH(c_custkey)
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES (
    "force_external_table_query_rewrite" = "TRUE"
)
AS
SELECT DISTINCT c_custkey, c_region FROM hive.ssb_1g_csv.customer; 

--mv_3_2
CREATE MATERIALIZED VIEW mv_3_2
DISTRIBUTED BY HASH(lo_orderdate)
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES (
    "force_external_table_query_rewrite" = "TRUE"
)
AS
SELECT l.lo_orderdate, l.lo_orderkey, mv.c_custkey, mv.c_region, SUM(l.lo_revenue)
FROM hive.ssb_1g_csv.lineorder l 
INNER JOIN mv_3_1 mv ON l.lo_custkey = mv.c_custkey
GROUP BY l.lo_orderkey, l.lo_orderdate, mv.c_custkey, mv.c_region;
```

### ケース4: データレイクでリアルタイムデータと履歴データのホットデータとコールドデータを分離する

過去3日以内に更新されたデータはStarRocksに直接書き込まれ、それより古いデータはチェックされてHiveにバッチ書き込みされますが、過去7日間のデータを含むクエリが存在する場合もあります。この場合、マテリアライズドビューを使用してシンプルなモデルを作成し、データを自動的に期限切れにすることができます。

```SQL
CREATE MATERIALIZED VIEW mv_4_1 
DISTRIBUTED BY HASH(lo_orderdate)
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
AS 
SELECT lo_orderkey, lo_orderdate, lo_revenue
FROM hive.ssb_1g_csv.lineorder
WHERE lo_orderdate <= current_date()
AND lo_orderdate >= date_add(current_date(), INTERVAL -4 DAY);
```

上位層のアプリケーションのロジックに基づいて、さらにビューやマテリアライズドビューを構築することができます。


---
displayed_sidebar: "Japanese"
---

# マテリアライズドビューを使用したデータレイククエリの高速化

このトピックでは、StarRocksの非同期マテリアライズドビューを使用して、データレイク内のクエリパフォーマンスを最適化する方法について説明します。

StarRocksは、データレイククエリの機能を提供しており、データの探索的なクエリや分析に非常に効果的です。ほとんどのシナリオでは、[データキャッシュ](../data_source/data_cache.md)を使用して、リモートストレージの揺れや大量のI/O操作によるパフォーマンスの低下を回避するためのブロックレベルのファイルキャッシュを提供できます。

ただし、データレイクからのデータを使用して複雑で効率的なレポートを構築したり、これらのクエリをさらに高速化する場合、パフォーマンスの課題に直面することがあります。非同期マテリアライズドビューを使用すると、レイク上のレポートやデータアプリケーションのクエリパフォーマンスを向上させるための高い並行性と優れたクエリパフォーマンスを実現できます。

## 概要

StarRocksは、Hiveカタログ、Icebergカタログ、Hudiカタログなどの外部カタログを基にした非同期マテリアライズドビューの構築をサポートしています。外部カタログベースのマテリアライズドビューは、次のシナリオで特に有用です。

- **データレイクレポートの透過的な高速化**

  データレイクレポートのクエリパフォーマンスを確保するために、データエンジニアは通常、データアナリストと緊密に連携して、レポートの高速化レイヤーの構築ロジックを調査する必要があります。高速化レイヤーをさらに更新する必要がある場合、構築ロジック、処理スケジュール、およびクエリステートメントを更新する必要があります。

  マテリアライズドビューのクエリ書き換え機能により、レポートの高速化をユーザーに透過的かつ感知できないようにすることができます。遅いクエリが特定された場合、データエンジニアは遅いクエリのパターンを分析し、必要に応じてマテリアライズドビューを作成することができます。アプリケーション側のクエリは、マテリアライズドビューによってインテリジェントに書き換えられ、ビジネスアプリケーションやクエリステートメントのロジックを変更することなくクエリパフォーマンスが迅速に改善されます。

- **過去のデータに関連するリアルタイムデータの増分計算**

  ビジネスアプリケーションでは、StarRocksのネイティブテーブル内のリアルタイムデータとデータレイク内の過去のデータを関連付けて増分計算する必要がある場合があります。このような場合、マテリアライズドビューを使用すると、簡単なソリューションを提供できます。たとえば、リアルタイムのファクトテーブルがStarRocksのネイティブテーブルであり、ディメンションテーブルがデータレイクに格納されている場合、マテリアライズドビューを構築して、ネイティブテーブルと外部データソースのテーブルを関連付けることで、増分計算を簡単に実行できます。

- **メトリックレイヤーの迅速な構築**

  メトリックの計算と処理は、データの高次元性を扱う場合に課題に直面することがあります。マテリアライズドビューを使用すると、データの事前集計とロールアップを実行することができるため、比較的軽量なメトリックレイヤーを作成できます。さらに、マテリアライズドビューは自動的に更新されるため、メトリックの計算の複雑さがさらに低減されます。

マテリアライズドビュー、データキャッシュ、およびStarRocksのネイティブテーブルは、すべてクエリパフォーマンスの大幅な向上を実現するための効果的な手段です。次の表は、それらの主な違いを比較しています。

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
      <td>クエリが自動的にデータキャッシュをトリガーします。</td>
      <td>リフレッシュタスクは自動的にトリガーされます。</td>
      <td>さまざまなインポート方法をサポートしますが、インポートタスクの手動メンテナンスが必要です。</td>
    </tr>
    <tr>
      <td><b>データキャッシュの粒度</b></td>
      <td><ul><li>ブロックレベルのデータキャッシュをサポート</li><li>LRUキャッシュのエビクションメカニズムに従う</li><li>計算結果はキャッシュされません</li></ul></td>
      <td>事前計算されたクエリ結果を格納します</td>
      <td>テーブルスキーマに基づいてデータを格納します</td>
    </tr>
    <tr>
      <td><b>クエリパフォーマンス</b></td>
      <td colspan="3" style={{textAlign: 'center'}} >データキャッシュ &le; マテリアライズドビュー = ネイティブテーブル</td>
    </tr>
    <tr>
      <td><b>クエリステートメント</b></td>
      <td><ul><li>データレイクに対するクエリステートメントを変更する必要はありません</li><li>クエリがキャッシュにヒットすると、計算が実行されます。</li></ul></td>
      <td><ul><li>データレイクに対するクエリステートメントを変更する必要はありません</li><li>クエリ書き換えを利用して事前計算結果を再利用します</li></ul></td>
      <td>ネイティブテーブルをクエリするためにクエリステートメントを変更する必要があります</td>
    </tr>
  </tbody>
</table>

<br />

データレイクから直接クエリを実行するか、データをネイティブテーブルにロードするかと比較して、マテリアライズドビューにはいくつかの独自の利点があります。

- **ローカルストレージの高速化**: マテリアライズドビューは、インデックス、パーティショニング、バケット分割、およびコロケートグループなどのStarRocksの高速化の利点をローカルストレージで活用することができます。これにより、データレイクからのデータクエリに比べてより優れたクエリパフォーマンスが実現されます。
- **ロードタスクのゼロメンテナンス**: マテリアライズドビューは、自動的なリフレッシュタスクによってデータを透過的に更新します。スケジュールされたデータの更新を実行するためにロードタスクをメンテナンスする必要はありません。さらに、Hiveカタログベースのマテリアライズドビューは、データの変更を検出し、パーティションレベルでの増分リフレッシュを実行することができます。
- **インテリジェントなクエリ書き換え**: クエリはマテリアライズドビューを透過的に書き換えることができます。アプリケーションが使用するクエリステートメントを変更する必要なく、即座に高速化の恩恵を受けることができます。

<br />

したがって、次のシナリオでは、マテリアライズドビューの使用をお勧めします。

- データキャッシュが有効になっているにもかかわらず、クエリパフォーマンスがクエリのレイテンシと並行性の要件を満たしていない場合。
- クエリに再利用可能なコンポーネント（固定の集計関数や結合パターンなど）が含まれている場合。
- データがパーティションで組織化されており、クエリが比較的高いレベルでの集計（たとえば、日ごとの集計）を行う場合。

<br />

次のシナリオでは、データキャッシュを介した高速化を優先することをお勧めします。

- クエリに再利用可能なコンポーネントがなく、データレイクから任意のデータをスキャンする可能性がある場合。
- リモートストレージには大きな変動や不安定性があり、アクセスに潜在的な影響がある場合。

## 外部カタログベースのマテリアライズドビューの作成

外部カタログのテーブルに対するマテリアライズドビューの作成は、StarRocksのネイティブテーブルに対するマテリアライズドビューの作成と同様です。適切なリフレッシュ戦略を設定し、外部カタログベースのマテリアライズドビューに対してクエリ書き換えを手動で有効にする必要があります。

### 適切なリフレッシュ戦略を選択する

現在、StarRocksはHudiカタログ、Icebergカタログ、JDBCカタログのパーティションレベルのデータ変更を検出することができません。したがって、タスクがトリガーされると完全なサイズのリフレッシュが実行されます。

Hiveカタログの場合、Hiveメタデータキャッシュのリフレッシュ機能を有効にすることで、パーティションレベルのデータ変更をStarRocksが検出できるようにすることができます。ただし、マテリアライズドビューのパーティショニングキーは、基になるテーブルのパーティショニングキーに含まれている必要があります。この機能を有効にすると、StarRocksは定期的にHiveメタストアサービス（HiveメタストアまたはAWS Glue）にアクセスし、最近クエリされたホットデータのメタデータ情報をチェックします。その結果、StarRocksは次のことができます。

- リフレッシュが必要なデータの変更のあるパーティションのみをリフレッシュし、リフレッシュによるリソース消費を削減します。

- クエリ書き換え中にデータの一貫性をある程度確保します。データレイクの基になるテーブルにデータの変更がある場合、クエリはマテリアライズドビューを使用して書き換えられません。

  > **注意**
  >
  > マテリアライズドビューを作成する際にプロパティ `mv_rewrite_staleness_second` を設定することで、一定レベルのデータの不整合を許容することもできます。詳細については、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

Hiveメタデータキャッシュのリフレッシュ機能を有効にするには、次のFEダイナミック設定項目を使用して、[ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)を実行します。

| **設定項目**                                                  | **デフォルト値**           | **説明**                                                     |
| ------------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| enable_background_refresh_connector_metadata                 | v3.0ではtrue、v2.5ではfalse | 定期的なHiveメタデータキャッシュのリフレッシュを有効にするかどうか。有効にすると、StarRocksはHiveクラスタのメタストア（HiveメタストアまたはAWS Glue）をポーリングし、頻繁にアクセスされるHiveカタログのキャッシュメタデータをリフレッシュしてデータの変更を検出します。trueはHiveメタデータキャッシュのリフレッシュを有効にし、falseは無効にします。 |
| background_refresh_metadata_interval_millis                  | 600000（10分）             | 2回の連続するHiveメタデータキャッシュリフレッシュ間のインターバル。単位：ミリ秒。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400（24時間）            | Hiveメタデータキャッシュリフレッシュタスクの有効期限。アクセスされたHiveカタログの場合、指定された時間以上アクセスされていない場合、StarRocksはキャッシュされたメタデータのリフレッシュを停止します。アクセスされていないHiveカタログの場合、StarRocksはキャッシュされたメタデータをリフレッシュしません。単位：秒。 |

### 外部カタログベースのマテリアライズドビューに対するクエリ書き換えを有効にする

デフォルトでは、StarRocksはHudiカタログ、Icebergカタログ、JDBCカタログに基づいて構築されたマテリアライズドビューに対するクエリ書き換えをサポートしていません。なぜなら、このシナリオではクエリ書き換えが結果の強い一貫性を保証できないためです。マテリアライズドビューをHiveカタログのテーブルに基づいて構築する場合、クエリ書き換えはデフォルトで有効になっています。

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

クエリ書き換えを伴うシナリオでは、非常に複雑なクエリステートメントを使用してマテリアライズドビューを構築する場合、クエリステートメントを分割し、ネストされた形式で複数の単純なマテリアライズドビューを構築することをお勧めします。ネストされたマテリアライズドビューはより柔軟で、さまざまなクエリパターンに対応できます。

## ベストプラクティス

実際のビジネスシナリオでは、監査ログや[ビッグクエリログ](../administration/monitor_manage_big_queries.md#analyze-big-query-logs)を分析することで、実行レイテンシが高くリソースを消費するクエリを特定することができます。さらに、[クエリプロファイル](../administration/query_profile.md)を使用して、クエリの遅延が発生している具体的なステージを特定することができます。次のセクションでは、マテリアライズドビューを使用してデータレイククエリのパフォーマンスを向上させる方法についての手順と例を提供します。

### ケース1：データレイクでの結合計算の高速化

マテリアライズドビューを使用して、データレイクでの結合クエリを高速化することができます。

次のHiveカタログ上のクエリが特に遅いとします。

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

クエリプロファイルを分析することで、これらのクエリの実行時間の大部分が`lineorder`テーブルと他のディメンションテーブルの`lo_orderdate`列を使用したハッシュ結合に費やされていることがわかるかもしれません。

ここで、Q1とQ2は`lineorder`と`dates`を結合した後に集計を実行し、Q3は`lineorder`、`dates`、`part`、および`supplier`を結合した後に集計を実行します。

したがって、`lineorder`、`dates`、`part`、および`supplier`を結合するマテリアライズドビューを構築するために、StarRocksの[ビューデルタ結合書き換え](./query_rewrite_with_materialized_views.md#view-delta-join-rewrite)機能を活用することができます。

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
    -- 外部カタログベースのマテリアライズドビューに対するクエリ書き換えを有効にします。
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

### ケース2：データレイクでの集計および結合による集計の高速化

マテリアライズドビューは、単一のテーブルに対する集計クエリ、および複数のテーブルを結合する集計クエリの両方を高速化するために使用することができます。

- 単一テーブルの集計クエリ

  単一のテーブルに対する典型的なクエリの場合、そのクエリプロファイルはAGGREGATEノードが多くの時間を消費していることを示すでしょう。一般的な集計演算子を使用してマテリアライズドビューを構築することができます。

  次のクエリが遅いとします。

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate limit 100;
  ```

  Q4は、ユニークな注文の日次数を計算します。ユニークな注文の数を計算する処理は計算コストが高いため、次の2つのタイプのマテリアライズドビューを作成して高速化することができます。

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
  -- lo_orderkeyはBIGINT型である必要があるため、クエリ書き換えに使用できます。
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  この文脈では、リミットとオーダーバイ句を含むマテリアライズドビューを作成しないでください。クエリ書き換えの制限事項については、[マテリアライズドビューによるクエリ書き換え - 制限事項](./query_rewrite_with_materialized_views.md#limitations)を参照してください。

- 複数テーブルの集計クエリ

  結合結果の集計を含むシナリオでは、テーブルを結合して得られた結果をさらに集計するための既存のマテリアライズドビュー上にネストされたマテリアライズドビューを作成することができます。たとえば、Case Oneの例に基づいて、Q1とQ2の集計パターンが類似しているため、次のマテリアライズドビューを作成してQ1とQ2を高速化することができます。

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

  もちろん、1つのマテリアライズドビュー内で結合と集計の計算を同時に実行することも可能です。このようなタイプのマテリアライズドビューは、クエリ書き換えの機会が少ない（特定の計算によるものなので）ですが、集計後のストレージスペースが通常よりも少なくなります。選択は具体的なユースケースに基づいて行うことができます。

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

### ケース3：データレイクでの集計結果の結合の高速化

一部のシナリオでは、まず1つのテーブルで集計計算を実行し、他のテーブルとの結合クエリを実行する必要があります。StarRocksのクエリ書き換え機能を最大限に活用するために、ネストされたマテリアライズドビューを構築することをお勧めします。たとえば：

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

Q5は、まず`customer`テーブルで集計クエリを実行し、`lineorder`テーブルとの結合および集計を実行します。同様のクエリでは、`c_region`および`lo_orderdate`に異なるフィルタが含まれる場合があります。クエリ書き換えの機能を活用するために、集計用のマテリアライズドビューと結合用のマテリアライズドビューの2つを作成することができます。

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

### ケース4：データレイクでのリアルタイムデータと過去のデータの分離

次のシナリオを考えてみましょう：過去3日間に更新されたデータは直接StarRocksに書き込まれ、より新しいデータはチェックされ、Hiveにバッチ書き込みされます。ただし、過去7日間のデータを含むクエリも存在する可能性があります。この場合、自動的にデータを期限切れにするためのシンプルなモデルとマテリアライズドビューを作成することができます。

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

上位アプリケーションのロジックに基づいてビューやマテリアライズドビューを構築することができます。

---
displayed_sidebar: Chinese
---

# データ湖クエリの高速化に物理ビューを使用する

この記事では、StarRocksの非同期物理ビューを使用して、データ湖内のクエリパフォーマンスを最適化する方法について説明します。

StarRocksは、データ湖クエリ機能を提供しており、データ湖内のデータを探索的なクエリ分析に非常に適しています。ほとんどの場合、[データキャッシュ](../data_source/data_cache.md)を使用することで、リモートストレージの遅延や大量のI/O操作によるパフォーマンスの低下を回避できます。

しかし、データ湖のデータを使用して複雑で効率的なレポートを構築したり、これらのクエリをさらに高速化したりする場合、パフォーマンスの課題に直面することがあります。非同期物理ビューを使用することで、データ湖内のレポートやアプリケーションの並行性を向上させ、パフォーマンスを向上させることができます。

## 概要

StarRocksは、Hiveカタログ、Icebergカタログ、Hudiカタログなどの外部カタログを使用して非同期物理ビューを構築することができます。外部カタログを使用した物理ビューは、次の場合に特に有用です。

- **データ湖レポートの透過的な高速化**

  データ湖レポートのクエリパフォーマンスを確保するために、データエンジニアは通常、データアナリストと緊密に連携して、アクセラレーションレイヤの構築ロジックを研究します。アクセラレーションレイヤの要件が更新されると、ビルドロジック、実行計画、クエリ文を適切に更新する必要があります。物理ビューのクエリリライト機能を使用することで、ユーザーはレポートのアクセラレーションプロセスを意識することなく利用できます。遅いクエリが特定された場合、データエンジニアは遅いクエリのパターンを分析し、必要に応じて物理ビューを作成することができます。その後、上位のクエリはスマートにリライトされ、物理ビューを透過的にアクセラレーションして、ビジネスアプリケーションのロジックやクエリ文を変更せずにクエリパフォーマンスを迅速に改善することができます。

- **リアルタイムデータとオフラインデータの関連付けによる増分計算**

  ビジネスアプリケーションでは、StarRocksのローカルテーブルのリアルタイムデータとデータ湖の履歴データを関連付けて増分計算を行う必要がある場合があります。このような場合、物理ビューは簡単な解決策を提供できます。たとえば、リアルタイムのファクトテーブルがStarRocksのローカルテーブルであり、ディメンションテーブルがデータ湖に格納されている場合、物理ビューを構築することで、ローカルテーブルと外部データソースのテーブルを関連付けて簡単に増分計算を行うことができます。

- **メトリックレイヤの迅速な構築**

  高次元のデータを処理する際に、メトリックの計算や処理には課題が発生する場合があります。物理ビューを使用してデータの事前集約やアップロードを行い、比較的軽量なメトリックレイヤを作成することができます。さらに、物理ビューの自動リフレッシュ機能を活用することで、メトリック計算の複雑さをさらに低減することができます。

物理ビュー、データキャッシュ、StarRocksのローカルテーブルは、クエリパフォーマンスを大幅に向上させるための効果的な方法です。以下の表は、それらの主な違いを比較しています。

<table class="comparison">
  <thead>
    <tr>
      <th>&nbsp;</th>
      <th>データキャッシュ</th>
      <th>物理ビュー</th>
      <th>ローカルテーブル</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>データのインポートと更新</b></td>
      <td>クエリがデータキャッシュを自動的にトリガーします</td>
      <td>自動的にリフレッシュタスクをトリガーします</td>
      <td>さまざまなインポート方法をサポートしますが、インポートタスクを手動で管理する必要があります</td>
    </tr>
    <tr>
      <td><b>データキャッシュの粒度</b></td>
      <td><ul><li>ブロックレベルのデータキャッシュをサポートします</li><li>LRUキャッシュの淘汰メカニズムに従います</li><li>計算結果はキャッシュされません</li></ul></td>
      <td>クエリの事前計算結果を保存します</td>
      <td>テーブル定義に基づいてデータを保存します</td>
    </tr>
    <tr>
      <td><b>クエリパフォーマンス</b></td>
      <td colspan="3" style={{textAlign: 'center'}} >データキャッシュ &le; 物理ビュー = ローカルテーブル</td>
    </tr>
    <tr>
      <td><b>クエリ文</b></td>
      <td><ul><li>データ湖のクエリ文を変更する必要はありません</li><li>クエリがキャッシュにヒットすると、現場で計算が行われます。</li></ul></td>
      <td><ul><li>データ湖のクエリ文を変更する必要はありません</li><li>クエリリライトを使用して事前計算結果を再利用します</li></ul></td>
      <td>ローカルテーブルをクエリするためにクエリ文を変更する必要があります</td>
    </tr>
  </tbody>
</table>

<br />

データ湖のデータを直接クエリしたり、データをローカルテーブルにインポートするよりも、物理ビューにはいくつかの独自の利点があります。

- **ローカルストレージの高速化**：物理ビューは、StarRocksのローカルストレージの利点を活用できます。インデックス、パーティショニング、バケット分割、Colocate Groupなどの機能を使用して、データ湖から直接クエリするよりも優れたクエリパフォーマンスを実現できます。
- **ロードタスクのメンテナンス不要**：物理ビューは、自動的にリフレッシュタスクを実行することでデータを透明に更新します。インポートタスクのメンテナンスは必要ありません。また、Hiveカタログベースの物理ビューでは、データの変更を検出し、パーティションレベルで増分リフレッシュを実行することができます。
- **スマートなクエリリライト**：クエリは物理ビューに透過的にリライトされ、アプリケーションで使用されるクエリ文を変更する必要なくクエリを高速化することができます。

<br />

したがって、物理ビューを使用することをお勧めします：

- データキャッシュを有効にしても、クエリの遅延や並行性の要件を満たすパフォーマンスが得られない場合。
- クエリに再利用可能な部分が含まれている場合、例えば、固定の集計方法、結合パターン。
- データがパーティション方式で組織化されており、クエリの集計レベルが高い場合（例：日次集計）。

<br />

次の場合は、データキャッシュを優先して使用することをお勧めします：

- クエリに再利用可能な部分がほとんどなく、データ湖の任意のデータに関連する可能性がある場合。
- リモートストレージに大きな変動や不安定性がある場合、アクセスに潜在的な影響がある場合。

## 外部カタログを使用した物理ビューの作成

外部カタログのテーブル上に物理ビューを作成する方法は、StarRocksのローカルテーブル上に物理ビューを作成する方法と似ています。使用しているデータソースに応じて適切なリフレッシュ戦略を設定し、外部カタログの物理ビューのクエリリライト機能を手動で有効にするだけです。

### 適切なリフレッシュ戦略の選択

現在、StarRocksはHudiカタログとJDBCカタログのパーティションレベルのデータ変更を検出することができません。したがって、リフレッシュタスクがトリガされると、フルリフレッシュが実行されます。

HiveカタログとIcebergカタログ（v3.1.4以降）では、パーティションレベルのデータ変更を検出することができます。したがって、StarRocksは次のことができます。

- データが変更されたパーティションのみをリフレッシュし、フルリフレッシュを回避してリフレッシュによるリソース消費を減らす。
- クエリリライト中にデータの整合性を一定程度保証する。データ湖の基本テーブルが変更された場合、クエリは物理ビューにリライトされません。

  > **注意**
  >
  > 物理ビューの作成時に、`mv_rewrite_staleness_second`プロパティを設定して、一定程度のデータ不整合を許容することもできます。詳細については、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

パーティションごとにリフレッシュする場合、物理ビューのパーティションキーは基本テーブルのパーティションキーに含まれている必要があります。

Hiveカタログの場合、Hiveメタデータキャッシュリフレッシュ機能を有効にすることができます。これにより、StarRocksは定期的にHiveメタデータストレージサービス（HMS）またはAWS Glueにアクセスし、最近アクセスされたホットデータのメタデータ情報を更新します。

Hiveメタデータキャッシュリフレッシュ機能を有効にするには、次のFE動的設定を使用して[ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)します。

| **設定名**                                                 | **デフォルト値**                      | **説明**                                                     |
| ------------------------------------------------------------ | ------------------------------- | ------------------------------------------------------------ |
| enable_background_refresh_connector_metadata                 | v3.0は`true`、v2.5は`false` | Hiveメタデータキャッシュの定期的なリフレッシュを有効にするかどうかを設定します。有効にすると、StarRocksはHiveクラスタのメタデータサービス（HMSまたはAWS Glue）をポーリングし、頻繁にアクセスされるHive外部データディレクトリのメタデータキャッシュをリフレッシュしてデータの更新を検知します。`true`は有効、`false`は無効を表します。 |
| background_refresh_metadata_interval_millis                  | 600000（10分）               | 2回のHiveメタデータキャッシュリフレッシュ間のインターバルです。単位：ミリ秒。         |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400（24時間）                | Hiveメタデータキャッシュリフレッシュタスクの有効期限です。アクセスされたHiveカタログがこの時間を超えてアクセスされない場合、そのカタログのメタデータキャッシュのリフレッシュを停止します。アクセスされていないHiveカタログについては、StarRocksはそのメタデータキャッシュをリフレッシュしません。単位：秒。 |

Icebergカタログの場合、v3.1.4以降、StarRocksはパーティションレベルのデータ変更を検出することができます。ただし、現時点ではIceberg V1テーブルのみをサポートしています。

### 外部カタログの物理ビューのクエリリライトを有効にする

データの強い整合性を保証できないため、StarRocksはデフォルトでHudi、Iceberg、JDBCカタログの物理ビューのクエリリライト機能を無効にしています。物理ビューを作成する際に、プロパティ`force_external_table_query_rewrite`を`true`に設定することで、この機能を有効にすることができます。Hiveカタログベースのテーブルから作成された物理ビューの場合、クエリリライト機能はデフォルトで有効になっています。クエリリライトに関与する場合、非常に複雑なクエリ文を使用して物理ビューを構築する場合、クエリ文を分割し、複数の単純な物理ビューをネストして構築することをお勧めします。ネストされた物理ビューはより柔軟で、より広範なクエリパターンに対応できます。

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
from `hudi_catalog`.`emp_db`.`emps_par_tbl`
where empid < 5;
```

## ベストプラクティス

実際のビジネスシナリオでは、監査ログや[ビッグクエリログ](../administration/monitor_manage_big_queries.md)を分析して、実行が遅い、リソース消費が高いクエリを特定することができます。また、[Query Profile](../administration/query_profile.md)を使用して、クエリが遅い特定のフェーズを正確に特定することもできます。以下の各セクションでは、マテリアライズドビューを使用してデータレイクのクエリパフォーマンスを向上させる方法と例を説明しています。

### ケース1：データレイクでのJoin計算の高速化

マテリアライズドビューを使用して、データレイク内のJoinクエリを高速化できます。

以下のHiveカタログ上のクエリが遅いと仮定します：

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

クエリのプロファイルを分析すると、`lineorder`テーブルと他のディメンションテーブルとの`lo_orderdate`列上のHash Joinに多くの実行時間が費やされていることがわかります。

ここで、Q1とQ2は`lineorder`と`dates`のJoin後に集約を行い、Q3は`lineorder`、`dates`、`part`、`supplier`のJoin後に集約を行います。

したがって、StarRocksの[View Delta Join改写](./query_rewrite_with_materialized_views.md#view-delta-join-改写)機能を利用して、`lineorder`、`dates`、`part`、`supplier`をJoinするマテリアライズドビューを構築できます。

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
    -- 外部キー制約を指定します。
    "foreign_key_constraints" = "
    hive.ssb_1g_csv.lineorder(lo_partkey) REFERENCES hive.ssb_1g_csv.part(p_partkey);
    hive.ssb_1g_csv.lineorder(lo_suppkey) REFERENCES hive.ssb_1g_csv.supplier(s_suppkey);
    hive.ssb_1g_csv.lineorder(lo_orderdate) REFERENCES hive.ssb_1g_csv.dates(d_datekey)",
    -- クエリ改写を有効にします。
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

### ケース2：データレイクでの集約とJoin後の集約計算の高速化

マテリアライズドビューは、単一テーブルの集約クエリにも、複数テーブルを含む場合にも、集約クエリを高速化するために使用できます。

- 単一テーブルの集約クエリ

  典型的な単一テーブルクエリに対して、Query ProfileがAGGREGATEノードに多くの時間がかかっていることを示している場合、一般的な集約演算子を使用してマテリアライズドビューを構築できます。以下のクエリが遅いとします：

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate limit 100;
  ```

  Q4は、日ごとのユニークな注文数を計算するクエリで、count distinctのコストが高いため、以下の2種類のマテリアライズドビューを作成できます：

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
  -- lo_orderkeyはBIGINT型である必要があります。これにより、クエリ改写に使用できます。
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  LIMITやORDER BY句を含むマテリアライズドビューを作成しないでください。これにより、改写が失敗する可能性があります。クエリ改写の制限についての詳細は、[マテリアライズドビュークエリ改写 - 制限](./query_rewrite_with_materialized_views.md#限制)を参照してください。

- 複数テーブルの集約クエリ

  Join結果の集約が関係するシナリオでは、既存の複数テーブルをJoinするマテリアライズドビューに対して、さらに集約を行うネストされたマテリアライズドビューを作成できます。例えば、ケース1の例に基づいて、Q1とQ2を高速化するために、以下のマテリアライズドビューを作成できます。これは、それらの集約パターンが似ているためです：

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

  もちろん、単一のマテリアライズドビューでJoinと集約計算を同時に実行することもできます。このタイプのマテリアライズドビューは、より具体的な計算を行うため、クエリ改写の機会が少なくなりますが、集約後はストレージスペースをより少なく占有します。実際のシナリオに基づいて選択できます。

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

### ケース3：データレイクでの集約後のJoin計算の高速化

場合によっては、まず一つのテーブルで集約計算を行い、その後で他のテーブルとJoinクエリを実行する必要があります。StarRocksのクエリ改写機能を最大限に活用するために、ネストされたマテリアライズドビューを構築することをお勧めします。例えば：

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
  lo_orderdate = '19970503'
```

Q5では、まず`customer`テーブルで集約を行い、その後`lineorder`テーブルでJoinと集約を行います。このようなクエリは、`c_region`や`lo_orderdate`に対する異なるフィルタ条件を含むことがあります。クエリ改写機能を利用するために、集約用とJoin用の2つのマテリアライズドビューを作成できます。

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

### ケース4：リアルタイムデータとデータレイクの履歴データの冷温分離

次のようなシナリオを想像してください：過去3日間の新しいデータは直接StarRocksに書き込まれ、3日前の古いデータは検証後にHiveにバッチ書き込みされます。しかし、クエリは過去7日間のデータを含む可能性があります。この場合、マテリアライズドビューを使用して、自動的にデータを期限切れにするシンプルなモデルを作成できます。

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

上位のビジネスロジックに基づいて、さらにビューまたはマテリアライズドビューを構築して計算をカプセル化することができます。

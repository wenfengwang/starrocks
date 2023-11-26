---
displayed_sidebar: "Japanese"
---

# クエリキャッシュ

クエリキャッシュは、StarRocksの強力な機能であり、集計クエリのパフォーマンスを大幅に向上させることができます。クエリキャッシュは、ローカル集計の中間結果をメモリに格納することで、以前のクエリと同一または類似の新しいクエリの不要なディスクアクセスと計算を回避することができます。クエリキャッシュにより、StarRocksは集計クエリの高速かつ正確な結果を提供し、時間とリソースを節約し、スケーラビリティを向上させることができます。クエリキャッシュは、多くのユーザーが大規模で複雑なデータセット上で類似のクエリを実行する高同時性のシナリオに特に有用です。

この機能はv2.5以降でサポートされています。

v2.5では、クエリキャッシュは単一のフラットテーブル上の集計クエリのみをサポートしています。v3.0以降、クエリキャッシュはスタースキーマで結合された複数のテーブル上の集計クエリもサポートしています。

## 適用シナリオ

次のシナリオでクエリキャッシュの使用をお勧めします。

- 個々のフラットテーブルまたはスタースキーマで結合された複数のテーブル上で頻繁に集計クエリを実行します。
- 集計クエリのほとんどがGROUP BYを使用しない集計クエリおよび低基数のGROUP BYを使用した集計クエリです。
- データは時間パーティションで追加モードでロードされ、アクセス頻度に基づいてホットデータとコールドデータに分類できます。

クエリキャッシュは、次の条件を満たすクエリをサポートしています。

- クエリエンジンがパイプラインであること。パイプラインエンジンを有効にするには、セッション変数`enable_pipeline_engine`を`true`に設定します。

  > **注意**
  >
  > 他のクエリエンジンはクエリキャッシュをサポートしていません。

- クエリはネイティブOLAPテーブル（v2.5以降）またはクラウドネイティブテーブル（v3.0以降）上の集計クエリです。クエリキャッシュは外部テーブル上のクエリをサポートしていません。クエリキャッシュは、計画が同期マテリアライズドビューへのアクセスを必要とするクエリをサポートしています。ただし、計画が非同期マテリアライズドビューへのアクセスを必要とするクエリはサポートしていません。

- クエリは個々のテーブルまたは結合された複数のテーブル上の集計クエリです。

  **注意**
  >
  > - クエリキャッシュはブロードキャストジョインとバケットシャッフルジョインをサポートしています。
  > - クエリキャッシュはJoin演算子を含む2つのツリー構造をサポートしています：Aggregation-JoinおよびJoin-Aggregation。Aggregation-Joinツリー構造ではシャッフルジョインはサポートされていませんが、Join-Aggregationツリー構造ではハッシュジョインはサポートされていません。

- クエリには`rand`、`random`、`uuid`、`sleep`などの非決定的な関数が含まれていません。

クエリキャッシュは、次のパーティションポリシーを使用するテーブル上のクエリをサポートしています：パーティションなし、複数列パーティション、および単一列パーティション。

## 機能の制約

- クエリキャッシュはパイプラインエンジンのテーブレット計算に基づいています。テーブレット計算は、パイプラインドライバが個々のテーブレットを一つずつ処理することを意味します。クエリごとに個々のBEが処理する必要があるテーブレットの数が、このクエリを実行するために呼び出されるパイプラインドライバの数以上である場合、クエリキャッシュは機能します。呼び出されるパイプラインドライバの数は、実際の並列度（DOP）を表します。テーブレットの数がパイプラインドライバの数よりも少ない場合、各パイプラインドライバは特定のテーブレットの一部のみを処理します。この状況では、テーブレット計算結果を生成することはできず、クエリキャッシュは機能しません。
- StarRocksでは、集計クエリは少なくとも4つのステージで構成されます。最初のステージのAggregateNodeによって生成されるテーブレットごとの計算結果は、OlapScanNodeとAggregateNodeが同じフラグメントからデータを計算する場合にのみキャッシュすることができます。他のステージのAggregateNodeによって生成されるテーブレットごとの計算結果はキャッシュすることはできません。一部のDISTINCT集計クエリでは、セッション変数`cbo_cte_reuse`が`true`に設定されている場合、データを生成するOlapScanNodeと生成されたデータを消費するステージ1のAggregateNodeが異なるフラグメントから計算され、ExchangeNodeによって接続されている場合、クエリキャッシュは機能しません。次の2つの例は、CTEの最適化が実行されるシナリオであり、クエリキャッシュは機能しないため、次のようになります。
  - 出力列は集計関数`avg(distinct)`を使用して計算されます。
  - 出力列は複数のDISTINCT集計関数によって計算されます。
- データが集計前にシャッフルされている場合、クエリキャッシュはそのデータ上のクエリを高速化することはできません。
- テーブルのGROUP BY列または重複排除列が高基数列である場合、そのテーブル上の集計クエリでは大きな結果が生成されます。この場合、クエリは実行時にクエリキャッシュをバイパスします。
- クエリキャッシュはBEが提供するわずかなメモリを占有して計算結果を保存します。クエリキャッシュのサイズはデフォルトで512 MBです。したがって、クエリキャッシュは大きなサイズのデータアイテムを保存するのには適していません。また、クエリキャッシュを有効にした後、キャッシュヒット率が低い場合、クエリのパフォーマンスが低下します。したがって、テーブレットごとに生成される計算結果のサイズが`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータで指定された閾値を超える場合、クエリキャッシュはクエリに対して機能せず、クエリはパススルーモードに切り替わります。

## 動作原理

クエリキャッシュが有効になっている場合、各BEはクエリのローカル集計を次の2つのステージに分割します。

1. テーブレットごとの集計

   BEは各テーブレットを個別に処理します。BEがテーブレットの処理を開始すると、まずクエリキャッシュを調査して、そのテーブレットの集計の中間結果がクエリキャッシュにあるかどうかを確認します。もしクエリキャッシュヒットがあれば、BEはクエリキャッシュから中間結果を直接取得します。クエリキャッシュミスの場合、BEはディスク上のデータにアクセスし、ローカル集計を実行して中間結果を計算します。BEがテーブレットの処理を終了すると、そのテーブレットの集計の中間結果をクエリキャッシュに格納します。

2. テーブレット間の集計

   BEはクエリに関与するすべてのテーブレットの中間結果を収集し、それらを最終結果にマージします。

   ![クエリキャッシュ - 動作原理 - 1](../assets/query_cache_principle-1.png)

将来的には、同じようなクエリが発行された場合、以前のクエリのキャッシュされた結果を再利用することができます。例えば、次の図に示すクエリでは、3つのテーブレット（テーブレット0から2）が関与しており、最初のテーブレット（テーブレット0）の中間結果は既にクエリキャッシュに格納されています。この例では、BEはディスク上のデータにアクセスする代わりに、クエリキャッシュからテーブレット0の結果を直接取得することができます。クエリキャッシュが完全にウォームアップされている場合、すべての3つのテーブレットの中間結果を含むことができるため、BEはディスク上のデータにアクセスする必要がありません。

![クエリキャッシュ - 動作原理 - 2](../assets/query_cache_principle-2.png)

余分なメモリを解放するために、クエリキャッシュは最も最近使用された（LRU）ベースのエビクションポリシーを採用してキャッシュエントリを管理します。このエビクションポリシーによれば、クエリキャッシュが定義されたサイズ（`query_cache_capacity`）を超えるメモリ量を占有する場合、最も最近使用されたキャッシュエントリがクエリキャッシュから削除されます。

> **注意**
>
> 将来的には、StarRocksはクエリキャッシュからエビクトされるキャッシュエントリを決定するためのTime to Live（TTL）ベースのエビクションポリシーもサポートします。

FEは、各クエリがクエリキャッシュを使用して高速化する必要があるかどうかを判断し、クエリの意味に影響を与えないトリビアルなリテラルの詳細を正規化します。

クエリキャッシュの悪いケースによって引き起こされるパフォーマンスのペナルティを防ぐために、BEは実行時にクエリキャッシュをバイパスするための適応ポリシーを採用しています。

## クエリキャッシュの有効化

このセクションでは、クエリキャッシュを有効化および設定するために使用されるパラメータとセッション変数について説明します。

### FEセッション変数

| **変数**                      | **デフォルト値** | **動的に設定可能** | **説明**                                                     |
| ----------------------------- | ----------------- | ------------------- | ------------------------------------------------------------ |
| enable_query_cache            | false             | Yes                 | クエリキャッシュを有効にするかどうかを指定します。有効な値は`true`および`false`です。`true`はこの機能を有効にし、`false`はこの機能を無効にします。クエリキャッシュが有効になっている場合、これはこのトピックの"[適用シナリオ](../using_starrocks/query_cache.md#適用シナリオ)"セクションで指定された条件を満たすクエリにのみ適用されます。 |
| query_cache_entry_max_bytes   | 4194304           | Yes                 | パススルーモードをトリガするための閾値を指定します。有効な値は`0`から`9223372036854775807`です。クエリが特定のテーブレットによってアクセスされる計算結果のバイト数または行数が、`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータで指定された閾値を超える場合、クエリはパススルーモードに切り替わります。<br />`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータが`0`に設定されている場合、関連するテーブレットから計算結果が生成されない場合でも、パススルーモードが使用されます。 |
| query_cache_entry_max_rows    | 409600            | Yes                 | 上記と同様です。                                              |

### BEパラメータ

BEの設定ファイル**be.conf**で次のパラメータを設定する必要があります。BEのパラメータを再構成した後は、BEを再起動して新しいパラメータ設定が有効になるようにする必要があります。

| **パラメータ**        | **必須** | **説明**                                                     |
| -------------------- | -------- | ------------------------------------------------------------ |
| query_cache_capacity | No       | クエリキャッシュのサイズを指定します。単位：バイト。デフォルトサイズは512 MBです。<br />各BEには独自のローカルクエリキャッシュがメモリにあり、自身のクエリキャッシュのみをポピュレートおよび調査します。<br />なお、クエリキャッシュのサイズは4 MB未満にすることはできません。BEのメモリ容量が期待するクエリキャッシュのサイズを供給するために不十分な場合は、BEのメモリ容量を増やすことができます。 |

## すべてのシナリオで最大のキャッシュヒット率を実現するために設計されました

クエリキャッシュは、次の3つのシナリオでもクエリが文字通り同一でなくても効果的です。

- 意味的に等価なクエリ
- スキャンされるパーティションが重複するクエリ
- 追加のみのデータ変更が行われるデータに対するクエリ（UPDATEやDELETE操作はなし）

### 意味的に等価なクエリ

2つのクエリが類似している場合、それらが文字通り同一である必要はありませんが、実行計画の意味的に等価なスニペットを含んでいる場合、それらは意味的に等価であり、互いの計算結果を再利用することができます。広義には、2つのクエリが同じソースからデータをクエリし、同じ計算方法を使用し、同じ実行計画を持つ場合、それらは意味的に等価であると評価されます。StarRocksは、次のルールを適用して2つのクエリが意味的に等価であるかどうかを評価します。

- 2つのクエリが複数の集計を含んでいる場合、最初の集計が意味的に等価である限り、それらは意味的に等価と評価されます。例えば、次の2つのクエリ、Q1とQ2は、複数の集計を含んでいますが、最初の集計が意味的に等価です。したがって、Q1とQ2は意味的に等価と評価されます。

  - Q1

    ```SQL
    SELECT
        (
            ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
    FROM
        (
            SELECT
                date_trunc('hour', ts) AS hour,
                k0,
                sum(v1) AS __c_0
            FROM
                t0
            WHERE
                ts between '2022-01-03 00:00:00'
                and '2022-01-03 23:59:59'
            GROUP BY
                date_trunc('hour', ts),
                k0
        ) AS t;
    ```

  - Q2

    ```SQL
    SELECT
        date_trunc('hour', ts) AS hour,
        k0,
        sum(v1) AS __c_0
    FROM
        t0
    WHERE
        ts between '2022-01-03 00:00:00'
        and '2022-01-03 23:59:59'
    GROUP BY
        date_trunc('hour', ts),
        k0
    ```

- 2つのクエリのいずれかが次のクエリタイプに属している場合、それらは意味的に等価と評価されます。ただし、HAVING句を含むクエリはHAVING句を含まないクエリと意味的に等価と評価されません。ただし、ORDER BY句またはLIMIT句の含まれていることは、2つのクエリが意味的に等価であるかどうかの評価に影響しません。

  - GROUP BY集計

    ```SQL
    SELECT <GroupByItems>, <AggFunctionItems> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>] 
    ```

    > **注意**
    >
    > 上記の例では、HAVING句はオプションです。

  - GROUP BY DISTINCT集計

    ```SQL
    SELECT DISTINCT <GroupByItems>, <Items> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    HAVING <HavingPredicate>
    ```

    > **注意**
    >
    > 上記の例では、HAVING句はオプションです。

  - 非GROUP BY集計

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

  - 非GROUP BY DISTINCT集計

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

- いずれかのクエリに`PartitionColumnRangePredicate`が含まれている場合、`PartitionColumnRangePredicate`は2つのクエリが意味的に等価であるかどうかを評価する前に削除されます。`PartitionColumnRangePredicate`は、パーティショニング列を参照する次のタイプの述語を指定します。

  - `col between v1 and v2`：パーティショニング列の値が[v1、v2]の範囲に含まれることを指定します。ここで、`v1`と`v2`は定数式です。
  - `v1 < col and col < v2`：パーティショニング列の値が(v1、v2)の範囲に含まれることを指定します。ここで、`v1`と`v2`は定数式です。
  - `v1 < col and col <= v2`：パーティショニング列の値が(v1、v2]の範囲に含まれることを指定します。ここで、`v1`と`v2`は定数式です。
  - `v1 <= col and col < v2`：パーティショニング列の値が[v1、v2)の範囲に含まれることを指定します。ここで、`v1`と`v2`は定数式です。
  - `v1 <= col and col <= v2`：パーティショニング列の値が[v1、v2]の範囲に含まれることを指定します。ここで、`v1`と`v2`は定数式です。

- 2つのクエリのSELECT句の出力列が再配置された後に同じである場合、2つのクエリは意味的に等価と評価されます。

- 2つのクエリのGROUP BY句の出力列が再配置された後に同じである場合、2つのクエリは意味的に等価と評価されます。

- 2つのクエリのWHERE句の残りの述語が`PartitionColumnRangePredicate`が削除された後に意味的に等価である場合、2つのクエリは意味的に等価と評価されます。

- 2つのクエリのHAVING句の述語が意味的に等価である場合、2つのクエリは意味的に等価と評価されます。

以下のテーブル`lineorder_flat`を例にして説明します。

```SQL
CREATE TABLE `lineorder_flat`
(
    `lo_orderdate` date NOT NULL COMMENT "",
    `lo_orderkey` int(11) NOT NULL COMMENT "",
    `lo_linenumber` tinyint(4) NOT NULL COMMENT "",
    `lo_custkey` int(11) NOT NULL COMMENT "",
    `lo_partkey` int(11) NOT NULL COMMENT "",
    `lo_suppkey` int(11) NOT NULL COMMENT "",
    `lo_orderpriority` varchar(100) NOT NULL COMMENT "",
    `lo_shippriority` tinyint(4) NOT NULL COMMENT "",
    `lo_quantity` tinyint(4) NOT NULL COMMENT "",
    `lo_extendedprice` int(11) NOT NULL COMMENT "",
    `lo_ordtotalprice` int(11) NOT NULL COMMENT "",
    `lo_discount` tinyint(4) NOT NULL COMMENT "",
    `lo_revenue` int(11) NOT NULL COMMENT "",
    `lo_supplycost` int(11) NOT NULL COMMENT "",
    `lo_tax` tinyint(4) NOT NULL COMMENT "",
    `lo_commitdate` date NOT NULL COMMENT "",
    `lo_shipmode` varchar(100) NOT NULL COMMENT "",
    `c_name` varchar(100) NOT NULL COMMENT "",
    `c_address` varchar(100) NOT NULL COMMENT "",
    `c_city` varchar(100) NOT NULL COMMENT "",
    `c_nation` varchar(100) NOT NULL COMMENT "",
    `c_region` varchar(100) NOT NULL COMMENT "",
    `c_phone` varchar(100) NOT NULL COMMENT "",
    `c_mktsegment` varchar(100) NOT NULL COMMENT "",
    `s_name` varchar(100) NOT NULL COMMENT "",
    `s_address` varchar(100) NOT NULL COMMENT "",
    `s_city` varchar(100) NOT NULL COMMENT "",
    `s_nation` varchar(100) NOT NULL COMMENT "",
    `s_region` varchar(100) NOT NULL COMMENT "",
    `s_phone` varchar(100) NOT NULL COMMENT "",
    `p_name` varchar(100) NOT NULL COMMENT "",
    `p_mfgr` varchar(100) NOT NULL COMMENT "",
    `p_category` varchar(100) NOT NULL COMMENT "",
    `p_brand` varchar(100) NOT NULL COMMENT "",
    `p_color` varchar(100) NOT NULL COMMENT "",
    `p_type` varchar(100) NOT NULL COMMENT "",
    `p_size` tinyint(4) NOT NULL COMMENT "",
    `p_container` varchar(100) NOT NULL COMMENT ""
)
ENGINE=OLAP 
DUPLICATE KEY(`lo_orderdate`, `lo_orderkey`)
COMMENT "olap"
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1 VALUES [('0000-01-01'), ('1993-01-01')),
PARTITION p2 VALUES [('1993-01-01'), ('1994-01-01')),
PARTITION p3 VALUES [('1994-01-01'), ('1995-01-01')),
PARTITION p4 VALUES [('1995-01-01'), ('1996-01-01')),
PARTITION p5 VALUES [('1996-01-01'), ('1997-01-01')),
PARTITION p6 VALUES [('1997-01-01'), ('1998-01-01')),
PARTITION p7 VALUES [('1998-01-01'), ('1999-01-01')))
DISTRIBUTED BY HASH(`lo_orderkey`)
PROPERTIES 
(
    "replication_num" = "1",
    "colocate_with" = "groupxx1",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "false",
    "compression" = "LZ4"
);
```

次の2つのクエリ、Q1とQ2は、テーブル`lineorder_flat`上で意味的に等価です。

1. SELECT文の出力列を再配置します。
2. GROUP BY句の出力列を再配置します。
3. ORDER BY句の出力列を削除します。
4. WHERE句の述語を再配置します。
5. `PartitionColumnRangePredicate`を追加します。

- Q1

  ```SQL
  SELECT sum(lo_revenue)), year(lo_orderdate) AS year,p_brand
  FROM lineorder_flat
  WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
  GROUP BY year,p_brand
  ORDER BY year,p_brand;
  ```

- Q2

  ```SQL
  SELECT year(lo_orderdate) AS year, p_brand, sum(lo_revenue))
  FROM lineorder_flat
  WHERE s_region = 'AMERICA' AND p_category = 'MFGR#12' AND 
     lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
  GROUP BY p_brand, year(lo_orderdate)
  ```

意味的な等価性は、クエリの物理的な計画に基づいて評価されます。したがって、クエリのリテラルの違いは、意味的な等価性の評価には影響しません。また、定数式はクエリの最適化中に削除され、`cast`式はクエリの最適化中に削除されます。したがって、これらの式は意味的な等価性の評価には影響しません。さらに、列やリレーションのエイリアスも意味的な等価性の評価には影響しません。

### スキャンされるパーティションが重複するクエリ

クエリキャッシュは、述語ベースのクエリ分割をサポートしています。

述語の意味に基づいてクエリを分割することで、部分的な計算結果の再利用を実現します。クエリにテーブルのパーティショニング列を参照する述語が含まれ、述語が値の範囲を指定する場合、StarRocksはテーブルのパーティショニングに基づいて範囲を複数の区間に分割することができます。各個別の区間の計算結果は、他のクエリによって個別に再利用されることができます。

次のテーブル`t0`を例に説明します。

```SQL
CREATE TABLE if not exists t0
(
    ts DATETIME NOT NULL,
    k0 VARCHAR(10) NOT NULL,
    k1 BIGINT NOT NULL,
    v1 DECIMAL64(7, 2) NOT NULL 
)
ENGINE=OLAP
DUPLICATE KEY(`ts`, `k0`, `k1`)
COMMENT "OLAP"
PARTITION BY RANGE(ts)
(
  START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 day) 
)
DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
PROPERTIES
(
    "replication_num" = "1", 
    "storage_format" = "default"
);
```

テーブル`t0`は日ごとにパーティション分割されており、列`ts`がテーブルのパーティショニング列です。次の4つのクエリのうち、Q2、Q3、およびQ4は、Q1のキャッシュされた計算結果の一部を再利用することができます。

- Q1

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1の述語`ts between '2022-01-02 12:30:00' and '2022-01-14 23:59:59'`で指定された値の範囲は、次の区間に分割することができます。

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  12. [2022-01-13 00:00:00, 2022-01-14 00:00:00),
  13. [2022-01-14 00:00:00, 2022-01-15 00:00:00),
  ```

- Q2

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-02 12:30:00' AND  ts < '2022-01-05 00:00:00'
  GROUP BY day;
  ```

  Q2は、Q1の次の区間内の計算結果を再利用することができます。

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ```

- Q3

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-01 12:30:00' AND  ts <= '2022-01-10 12:00:00'
  GROUP BY day;
  ```

  Q3は、Q1の次の区間内の計算結果を再利用することができます。

  ```SQL
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  8. [2022-01-09 00:00:00, 2022-01-10 00:00:00),
  ```

- Q4

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' and '2022-01-02 23:59:59'
  GROUP BY day;
  ```

  Q4は、Q1の次の区間内の計算結果を再利用することができます。

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

部分的な計算結果の再利用のサポートは、使用するパーティショニングポリシーによって異なります。詳細は以下の表を参照してください。

| **パーティショニングポリシー**     | **部分的な計算結果の再利用のサポート** |
| ---------------------------------- | ---------------------------------------- |
| パーティションなし                   | サポートされていません                     |
| 複数列パーティション                 | サポートされていません<br />**注意**<br />将来的にはサポートされる可能性があります。 |
| 単一列パーティション                 | サポートされています                       |

### 追加のみのデータ変更が行われるデータに対するクエリ

クエリキャッシュは、マルチバージョンキャッシングをサポートしています。
データのロードが行われると、新しいバージョンのタブレットが生成されます。その結果、以前のバージョンのタブレットから生成されたキャッシュされた計算結果は古くなり、最新のタブレットのバージョンに遅れます。このような状況では、マルチバージョンキャッシュメカニズムは、クエリキャッシュに保存された古い結果とディスク上に保存されたインクリメンタルなタブレットのバージョンを最終結果にマージし、新しいクエリが最新のタブレットのバージョンを持つことができるようにします。マルチバージョンキャッシュは、テーブルの種類、クエリの種類、およびデータの更新の種類によって制約されます。

マルチバージョンキャッシュのサポートは、テーブルの種類とクエリの種類によって異なります。次の表に示すように、次のテーブルの種類とクエリの種類に対するマルチバージョンキャッシュのサポートが異なります。

| テーブルの種類 | クエリの種類 | マルチバージョンキャッシュのサポート |
| -------------- | ------------ | ------------------------------------ |
| 重複キーテーブル | <ul><li>ベーステーブルのクエリ</li><li>同期マテリアライズドビューのクエリ</li></ul> | <ul><li>ベーステーブルのクエリ：インクリメンタルなタブレットのバージョンにデータ削除レコードが含まれていない場合を除き、すべての状況でサポートされます。</li><li>同期マテリアライズドビューのクエリ：GROUP BY、HAVING、またはWHERE句のクエリが集計列を参照している場合を除き、すべての状況でサポートされます。</li></ul> |
| 集約テーブル | ベーステーブルのクエリまたは同期マテリアライズドビューのクエリ | 次のすべての状況でサポートされます。ベーステーブルのスキーマには集約関数 `replace` が含まれていない場合。クエリのGROUP BY、HAVING、またはWHERE句が集計列を参照していない場合。インクリメンタルなタブレットのバージョンにデータ削除レコードが含まれていない場合。 |
| ユニークキーテーブル | N/A | サポートされていません。ただし、クエリキャッシュはサポートされています。 |
| プライマリキーテーブル | N/A | サポートされていません。ただし、クエリキャッシュはサポートされています。 |

データの更新の種類がマルチバージョンキャッシュに与える影響は次のとおりです。

- データの削除

  インクリメンタルなタブレットのバージョンに削除操作が含まれている場合、マルチバージョンキャッシュは機能しません。

- データの挿入

  - タブレットの空のバージョンが生成された場合、クエリキャッシュの既存のデータは有効であり、取得することができます。
  - タブレットの非空のバージョンが生成された場合、クエリキャッシュの既存のデータは有効ですが、そのバージョンはタブレットの最新バージョンに遅れています。この場合、StarRocksは既存データから最新のタブレットのバージョンまでのインクリメンタルデータを読み取り、既存データとインクリメンタルデータをマージし、マージされたデータをクエリキャッシュに格納します。

- スキーマの変更とタブレットの切り捨て

  テーブルのスキーマが変更されるか、テーブルの特定のタブレットが切り捨てられる場合、テーブルの新しいタブレットが生成されます。その結果、クエリキャッシュのタブレットの既存データは無効になります。

## メトリクス

クエリキャッシュが機能するクエリのプロファイルには、`CacheOperator` の統計情報が含まれます。

クエリのソースプランに `OlapScanOperator` が含まれる場合、`OlapScanOperator` と集約演算子の名前は `ML_` で接頭辞が付けられ、パイプラインで `MultilaneOperator` を使用してタブレットごとの計算を実行することを示します。`CacheOperator` は、Passthrough、Populate、およびProbeモードでクエリキャッシュが実行される方法を制御するロジックを処理するために `ML_CONJUGATE_AGGREGATE` の前に挿入されます。クエリのプロファイルには、クエリキャッシュの使用状況を理解するのに役立つ次の `CacheOperator` のメトリクスが含まれています。

| メトリクス名               | 説明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| CachePassthroughBytes     | Passthroughモードで生成されたバイト数。                      |
| CachePassthroughChunkNum  | Passthroughモードで生成されたチャンク数。                     |
| CachePassthroughRowNum    | Passthroughモードで生成された行数。                           |
| CachePassthroughTabletNum | Passthroughモードで生成されたタブレット数。                    |
| CachePassthroughTime:     | Passthroughモードでの計算時間。                               |
| CachePopulateBytes        | Populateモードで生成されたバイト数。                          |
| CachePopulateChunkNum     | Populateモードで生成されたチャンク数。                         |
| CachePopulateRowNum       | Populateモードで生成された行数。                              |
| CachePopulateTabletNum    | Populateモードで生成されたタブレット数。                       |
| CachePopulateTime         | Populateモードでの計算時間。                                  |
| CacheProbeBytes           | Probeモードでキャッシュヒット時に生成されたバイト数。         |
| CacheProbeChunkNum        | Probeモードでキャッシュヒット時に生成されたチャンク数。        |
| CacheProbeRowNum          | Probeモードでキャッシュヒット時に生成された行数。              |
| CacheProbeTabletNum       | Probeモードでキャッシュヒット時に生成されたタブレット数。       |
| CacheProbeTime            | Probeモードでの計算時間。                                     |

`CachePopulate`*`XXX`* メトリクスは、クエリキャッシュが更新されたクエリキャッシュミスに関する統計情報を提供します。

`CachePassthrough`*`XXX`* メトリクスは、クエリキャッシュが更新されなかったクエリキャッシュミスに関する統計情報を提供します。これは、パーティションごとの計算結果のサイズが大きいためです。

`CacheProbe`*`XXX`* メトリクスは、クエリキャッシュヒットに関する統計情報を提供します。

マルチバージョンキャッシュメカニズムでは、`CachePopulate` メトリクスと `CacheProbe` メトリクスには同じタブレットの統計情報が含まれる場合があります。たとえば、StarRocksは各タブレットのデータを計算する際に、タブレットの過去のバージョンで生成された計算結果にヒットします。この場合、StarRocksは過去のバージョンから最新のバージョンまでのインクリメンタルデータを読み取り、データを計算し、インクリメンタルデータをキャッシュされたデータとマージします。マージ後の計算結果のサイズが `query_cache_entry_max_bytes` パラメータまたは `query_cache_entry_max_rows` パラメータで指定された閾値を超えない場合、タブレットの統計情報は `CachePopulate` メトリクスに収集されます。それ以外の場合、タブレットの統計情報は `CachePassthrough` メトリクスに収集されます。

## RESTful API操作

- `metrics |grep query_cache`

  このAPI操作は、クエリキャッシュに関連するメトリクスをクエリするために使用されます。

  ```shell
  curl -s  http://<be_host>:<be_http_port>/metrics |grep query_cache
  
  # TYPE starrocks_be_query_cache_capacity gauge
  starrocks_be_query_cache_capacity 536870912
  # TYPE starrocks_be_query_cache_hit_count gauge
  starrocks_be_query_cache_hit_count 5084393
  # TYPE starrocks_be_query_cache_hit_ratio gauge
  starrocks_be_query_cache_hit_ratio 0.984098
  # TYPE starrocks_be_query_cache_lookup_count gauge
  starrocks_be_query_cache_lookup_count 5166553
  # TYPE starrocks_be_query_cache_usage gauge
  starrocks_be_query_cache_usage 0
  # TYPE starrocks_be_query_cache_usage_ratio gauge
  starrocks_be_query_cache_usage_ratio 0.000000
  ```

- `api/query_cache/stat`

  このAPI操作は、クエリキャッシュの使用状況をクエリするために使用されます。

  ```shell
  curl  http://<be_host>:<be_http_port>/api/query_cache/stat
  {
      "capacity": 536870912,
      "usage": 0,
      "usage_ratio": 0.0,
      "lookup_count": 5025124,
      "hit_count": 4943720,
      "hit_ratio": 0.983800598751394
  }
  ```

- `api/query_cache/invalidate_all`

  このAPI操作は、クエリキャッシュをクリアするために使用されます。

  ```shell
  curl  -XPUT http://<be_host>:<be_http_port>/api/query_cache/invalidate_all
  
  {
      "status": "OK"
  }
  ```

前述のAPI操作のパラメータは次のとおりです。

- `be_host`：BEが存在するノードのIPアドレスです。
- `be_http_port`：BEが存在するノードのHTTPポート番号です。

## 注意事項

- StarRocksは、初めて実行されるクエリの計算結果をクエリキャッシュに格納する必要があります。そのため、クエリのパフォーマンスは予想よりもわずかに低下し、クエリのレイテンシが増加する場合があります。
- クエリキャッシュサイズを大きく設定すると、BEでのクエリ評価に割り当てることができるメモリ量が減少します。クエリ評価に割り当てるメモリ容量の1/6を超えないようにすることをお勧めします。
- 処理する必要のあるタブレットの数が `pipeline_dop` の値よりも小さい場合、クエリキャッシュは機能しません。クエリキャッシュを有効にするには、`pipeline_dop` を `1` のような小さい値に設定することができます。StarRocksは、v3.0以降、クエリの並列処理に基づいてこのパラメータを自動的に調整します。

## 例

### データセット

1. StarRocksクラスタにログインし、宛先データベースに移動し、次のコマンドを実行して `t0` という名前のテーブルを作成します。

   ```SQL
   CREATE TABLE t0
   (
         `ts` datetime NOT NULL COMMENT "",
         `k0` varchar(10) NOT NULL COMMENT "",
         `k1` char(6) NOT NULL COMMENT "",
         `v0` bigint(20) NOT NULL COMMENT "",
         `v1` decimal64(7, 2) NOT NULL COMMENT ""
   )
   ENGINE=OLAP 
   DUPLICATE KEY(`ts`, `k0`, `k1`)
   COMMENT "OLAP"
   PARTITION BY RANGE(`ts`)
   (
       START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 DAY)
   )
   DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
   PROPERTIES
   (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "enable_persistent_index" = "false"
   );
   ```

2. 次のデータレコードを `t0` に挿入します。

   ```SQL
   INSERT INTO t0
   VALUES
       ('2022-01-11 20:42:26', 'n4AGcEqYp', 'hhbawx', '799393174109549', '8029.42'),
       ('2022-01-27 18:17:59', 'i66lt', 'mtrtzf', '100400167', '10000.88'),
       ('2022-01-28 20:10:44', 'z6', 'oqkeun', '-58681382337', '59881.87'),
       ('2022-01-29 14:54:31', 'qQ', 'dzytua', '-19682006834', '43807.02'),
       ('2022-01-31 08:08:11', 'qQ', 'dzytua', '7970665929984223925', '-8947.74'),
       ('2022-01-15 00:40:58', '65', 'hhbawx', '4054945', '156.56'),
       ('2022-01-24 16:17:51', 'onqR3JsK1', 'udtmfp', '-12962', '-72127.53'),
       ('2022-01-01 22:36:24', 'n4AGcEqYp', 'fabnct', '-50999821', '17349.85'),
       ('2022-01-21 08:41:50', 'Nlpz1j3h', 'dzytua', '-60162', '287.06'),
       ('2022-01-30 23:44:55', '', 'slfght', '62891747919627339', '8014.98'),
       ('2022-01-18 19:14:28', 'z6', 'dzytua', '-1113001726', '73258.24'),
       ('2022-01-30 14:54:59', 'z6', 'udtmfp', '111175577438857975', '-15280.41'),
       ('2022-01-08 22:08:26', 'z6', 'ympyls', '3', '2.07'),
       ('2022-01-03 08:17:29', 'Nlpz1j3h', 'udtmfp', '-234492', '217.58'),
       ('2022-01-27 07:28:47', 'Pc', 'cawanm', '-1015', '-20631.50'),
       ('2022-01-17 14:07:47', 'Nlpz1j3h', 'lbsvqu', '2295574006197343179', '93768.75'),
       ('2022-01-31 14:00:12', 'onqR3JsK1', 'umlkpo', '-227', '-66199.05'),
       ('2022-01-05 20:31:26', '65', 'lbsvqu', '684307', '36412.49'),
       ('2022-01-06 00:51:34', 'z6', 'dzytua', '11700309310', '-26064.10'),
       ('2022-01-26 02:59:00', 'n4AGcEqYp', 'slfght', '-15320250288446', '-58003.69'),
       ('2022-01-05 03:26:26', 'z6', 'cawanm', '19841055192960542', '-5634.36'),
       ('2022-01-17 08:51:23', 'Pc', 'ghftus', '35476438804110', '13625.99'),
       ('2022-01-30 18:56:03', 'n4AGcEqYp', 'lbsvqu', '3303892099598', '8.37'),
       ('2022-01-22 14:17:18', 'i66lt', 'umlkpo', '-27653110', '-82306.25'),
       ('2022-01-02 10:25:01', 'qQ', 'ghftus', '-188567166', '71442.87'),
       ('2022-01-30 04:58:14', 'Pc', 'ympyls', '-9983', '-82071.59'),
       ('2022-01-05 00:16:56', '7Bh', 'hhbawx', '43712', '84762.97'),
       ('2022-01-25 03:25:53', '65', 'mtrtzf', '4604107', '-2434.69'),
       ('2022-01-27 21:09:10', '65', 'udtmfp', '476134823953365199', '38736.04'),
       ('2022-01-11 13:35:44', '65', 'qmwhvr', '1', '0.28'),
       ('2022-01-03 19:13:07', '', 'lbsvqu', '11', '-53084.04'),
       ('2022-01-20 02:27:25', 'i66lt', 'umlkpo', '3218824416', '-71393.20'),
       ('2022-01-04 04:52:36', '7Bh', 'ghftus', '-112543071', '-78377.93'),
       ('2022-01-27 18:27:06', 'Pc', 'umlkpo', '477', '-98060.13'),
       ('2022-01-04 19:40:36', '', 'udtmfp', '433677211', '-99829.94'),
       ('2022-01-20 23:19:58', 'Nlpz1j3h', 'udtmfp', '361394977', '-19284.18'),
       ('2022-01-05 02:17:56', 'Pc', 'oqkeun', '-552390906075744662', '-25267.92'),
       ('2022-01-02 16:14:07', '65', 'dzytua', '132', '2393.77'),
       ('2022-01-28 23:17:14', 'z6', 'umlkpo', '61', '-52028.57'),
       ('2022-01-12 08:05:44', 'qQ', 'hhbawx', '-9579605666539132', '-87801.81'),
       ('2022-01-31 19:48:22', 'z6', 'lbsvqu', '9883530877822', '34006.42'),
       ('2022-01-11 20:38:41', '', 'piszhr', '56108215256366', '-74059.80'),
       ('2022-01-01 04:15:17', '65', 'cawanm', '-440061829443010909', '88960.51'),
       ('2022-01-05 07:26:09', 'qQ', 'umlkpo', '-24889917494681901', '-23372.12'),
       ('2022-01-29 18:13:55', 'Nlpz1j3h', 'cawanm', '-233', '-24294.42'),
       ('2022-01-10 00:49:45', 'Nlpz1j3h', 'ympyls', '-2396341', '77723.88'),
       ('2022-01-29 08:02:58', 'n4AGcEqYp', 'slfght', '45212', '93099.78'),
       ('2022-01-28 08:59:21', 'onqR3JsK1', 'oqkeun', '76', '-78641.65'),
       ('2022-01-26 14:29:39', '7Bh', 'umlkpo', '176003552517', '-99999.96'),
       ('2022-01-03 18:53:37', '7Bh', 'piszhr', '3906151622605106', '55723.01'),
       ('2022-01-04 07:08:19', 'i66lt', 'ympyls', '-240097380835621', '-81800.87'),
       ('2022-01-28 14:54:17', 'Nlpz1j3h', 'slfght', '-69018069110121', '90533.64'),
       ('2022-01-22 07:48:53', 'Pc', 'ympyls', '22396835447981344', '-12583.39'),
       ('2022-01-22 07:39:29', 'Pc', 'uqkghp', '10551305', '52163.82'),
       ('2022-01-08 22:39:47', 'Nlpz1j3h', 'cawanm', '67905472699', '87831.30'),
       ('2022-01-05 14:53:34', '7Bh', 'dzytua', '-779598598706906834', '-38780.41'),
       ('2022-01-30 17:34:41', 'onqR3JsK1', 'oqkeun', '346687625005524', '-62475.31'),
       ('2022-01-29 12:14:06', '', 'qmwhvr', '3315', '22076.88'),
       ('2022-01-05 06:47:04', 'Nlpz1j3h', 'udtmfp', '-469', '42747.17'),
       ('2022-01-19 15:20:20', '7Bh', 'lbsvqu', '347317095885', '-76393.49'),
       ('2022-01-08 16:18:22', 'z6', 'fghmcd', '2', '90315.60'),
       ('2022-01-02 00:23:06', 'Pc', 'piszhr', '-3651517384168400', '58220.34'),
       ('2022-01-12 08:23:31', 'onqR3JsK1', 'udtmfp', '5636394870355729225', '33224.25'),
       ('2022-01-28 10:46:44', 'onqR3JsK1', 'oqkeun', '-28102078612755', '6469.53'),
       ('2022-01-23 23:16:11', 'onqR3JsK1', 'ghftus', '-707475035515433949', '63422.66'),
       ('2022-01-03 05:32:31', 'z6', 'hhbawx', '-45', '-49680.52'),
       ('2022-01-27 03:24:33', 'qQ', 'qmwhvr', '375943906057539870', '-66092.96'),
       ('2022-01-25 20:07:22', '7Bh', 'slfght', '1', '72440.21'),
       ('2022-01-04 16:07:24', 'qQ', 'uqkghp', '751213107482249', '16417.31'),
       ('2022-01-23 19:22:00', 'Pc', 'hhbawx', '-740731249600493', '88439.40'),
       ('2022-01-05 09:04:20', '7Bh', 'cawanm', '23602', '302.44');
   ```

### クエリの例

このセクションのクエリキャッシュ関連メトリクスの統計情報は例であり、参考のために提供されています。

#### クエリキャッシュはステージ1のローカル集計に対して機能します

これには、次の3つの状況が含まれます。

- クエリが単一のタブレットにのみアクセスする場合。
- クエリが自体が共有グループを構成するテーブルの複数のパーティションから複数のタブレットにアクセスし、集計のためにデータをシャッフルする必要がない場合。
- クエリがテーブルの同じパーティションから複数のタブレットにアクセスし、集計のためにデータをシャッフルする必要がない場合。

クエリの例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    k0,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    k0
```

以下の図は、クエリプロファイルのクエリキャッシュ関連メトリクスを示しています。

![Query Cache - Stage 1 - Metrics](../assets/query_cache_stage1_agg_with_cache_jp.png)

#### クエリキャッシュはステージ1のリモート集計に対して機能しません

ステージ1で複数のタブレットの集計が強制的に実行される場合、データはまずシャッフルされ、その後集計されます。

クエリの例：

```SQL
SET new_planner_agg_stage = 1;

SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

#### クエリキャッシュはステージ2のローカル集計に対して機能します

これには、次の3つの状況が含まれます。

- クエリのステージ2の集計は同じタイプのデータを比較するためにコンパイルされます。最初の集計はローカル集計です。最初の集計が完了した後、最初の集計から生成された結果を計算して、2番目の集計（グローバル集計）を実行します。
- クエリが SELECT DISTINCT クエリである場合。
- クエリに次のいずれかの DISTINCT 集計関数が含まれている場合：`sum(distinct)`、`count(distinct)`、`avg(distinct)`。通常、このようなクエリではステージ3またはステージ4で集計が実行されますが、クエリでステージ2で集計を強制的に実行するために `set new_planner_agg_stage = 1` を実行することもできます。クエリに `avg(distinct)` が含まれていてステージで集計を実行する場合、CTEの最適化を無効にするために `set cbo_cte_reuse = false` も実行する必要があります。

クエリの例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

以下の図は、クエリプロファイルのクエリキャッシュ関連メトリクスを示しています。

![Query Cache - Stage 2 - Metrics](../assets/query_cache_stage2_agg_with_cache_jp.png)

#### クエリキャッシュはステージ3のローカル集計に対して機能します

クエリは、単一の DISTINCT 集計関数を含む GROUP BY 集計クエリです。

サポートされている DISTINCT 集計関数は `sum(distinct)`、`count(distinct)`、`avg(distinct)` です。

> **注意**
>
> クエリに `avg(distinct)` が含まれる場合、`set cbo_cte_reuse = false` を実行して CTE の最適化を無効にする必要があります。

クエリの例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd;
```

以下の図は、クエリプロファイルのクエリキャッシュ関連メトリクスを示しています。
![Query Cache - ステージ3 - メトリクス](../assets/query_cache_stage3_agg_with_cache_jp.png)

#### クエリキャッシュはステージ4のローカル集計に対して機能します

クエリは、単一のDISTINCT集計関数を含むGROUP BYのない集計クエリです。このようなクエリには、重複データを削除する古典的なクエリが含まれます。

クエリの例：

```SQL
SELECT
    count(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
```

以下の図は、クエリプロファイルでのクエリキャッシュに関連するメトリクスを示しています。

![Query Cache - ステージ4 - メトリクス](../assets/query_cache_stage4_agg_with_cache_jp.png)

#### キャッシュされた結果は、最初の集計が意味的に等価な2つのクエリに再利用されます

以下の2つのクエリ、Q1とQ2を例にして説明します。Q1とQ2はどちらも複数の集計を含んでいますが、最初の集計は意味的に等価です。そのため、Q1とQ2は意味的に等価として評価され、クエリキャッシュに保存された計算結果を相互に再利用することができます。

- Q1

  ```SQL
  SELECT
      (
          ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
  FROM
      (
          SELECT
              date_trunc('hour', ts) AS hour,
              k0,
              sum(v1) AS __c_0
          FROM
              t0
          WHERE
              ts between '2022-01-03 00:00:00'
              and '2022-01-03 23:59:59'
          GROUP BY
              date_trunc('hour', ts),
              k0
      ) AS t;
  ```

- Q2

  ```SQL
  SELECT
      date_trunc('hour', ts) AS hour,
      k0,
      sum(v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59'
  GROUP BY
      date_trunc('hour', ts),
      k0
  ```

以下の図は、Q1の`CachePopulate`メトリクスを示しています。

![Query Cache - Q1 - メトリクス](../assets/query_cache_reuse_Q1_jp.png)

以下の図は、Q2の`CacheProbe`メトリクスを示しています。

![Query Cache - Q2 - メトリクス](../assets/query_cache_reuse_Q2_jp.png)

#### CTEの最適化が有効なDISTINCTクエリにはクエリキャッシュは機能しません

`set cbo_cte_reuse = true`を実行してCTEの最適化を有効にした後、DISTINCT集計関数を含む特定のクエリの計算結果はキャッシュされません。以下にいくつかの例を示します：

- クエリに単一のDISTINCT集計関数`avg(distinct)`が含まれる場合：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 1](../assets/query_cache_distinct_with_cte_Q1_jp.png)

- クエリに同じ列を参照する複数のDISTINCT集計関数が含まれる場合：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0,
      sum(distinct v1) AS __c_1,
      count(distinct v1) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 2](../assets/query_cache_distinct_with_cte_Q2_jp.png)

- クエリに異なる列を参照する複数のDISTINCT集計関数が含まれる場合：

  ```SQL
  SELECT
      sum(distinct v1) AS __c_1,
      count(distinct v0) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_jp.png)

## ベストプラクティス

テーブルを作成する際には、次のような適切なパーティションの設定と分散方法を指定してください：

- パーティション列として単一のDATE型の列を選択してください。テーブルに複数のDATE型の列が含まれる場合は、データが増分的にインジェストされるにつれて値がロールフォワードし、クエリの興味深い時間範囲を定義するために使用される列を選択してください。
- 適切なパーティション幅を選択してください。最新のパーティションは最新のデータによって変更される場合があります。そのため、最新のパーティションを含むキャッシュエントリは不安定であり、無効になる可能性があります。
- テーブル作成文の分散記述には、数十個のバケット番号を指定してください。バケット番号が極端に小さい場合、BEが処理する必要のあるタブレットの数が`pipeline_dop`の値よりも少ない場合、クエリキャッシュは効果を発揮できません。

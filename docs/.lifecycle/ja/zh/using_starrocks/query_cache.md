---
displayed_sidebar: Chinese
---

# クエリキャッシュ

StarRocksのクエリキャッシュ機能は、集計クエリのパフォーマンスを大幅に向上させることができます。クエリキャッシュを有効にすると、StarRocksは集計クエリの処理ごとに、ローカルでの集計中間結果をメモリにキャッシュします。そのため、後続の同じまたは類似の集計クエリが届いた場合、StarRocksはクエリキャッシュから一致する集計結果を直接取得することができます。ディスクからデータを読み込んで計算する必要がなくなり、クエリの時間とリソースのコストを大幅に節約し、クエリのスケーラビリティを向上させることができます。大量のユーザーが同じまたは類似のクエリを複雑な大規模データセットに対して同時に実行するような高並行のシナリオでは、クエリキャッシュの利点が特に顕著です。

この機能は2.5バージョンからサポートされています。

2.5バージョンでは、クエリキャッシュはワイドテーブルモデルの単一テーブル集計クエリのみをサポートしていました。3.0バージョン以降、ワイドテーブルモデルの単一テーブル集計クエリに加えて、スターモデルモデルのシンプルな複数テーブルJOINの集計クエリもサポートしています。

## 適用シナリオ

クエリキャッシュが有効になる典型的な適用シナリオは次の特徴を持っています：

- クエリは主にワイドテーブルモデルの単一テーブル集計クエリまたはスターモデルモデルのシンプルな複数テーブルJOINの集計クエリです。
- 集計クエリはGROUP BYを使用しない集計または低基数のGROUP BY集計が主です。
- クエリのデータは時間によるパーティション分割で追加され、異なる時間のパーティションにアクセスすると冷たい/ホットなアクセスパターンが現れます。

現在、クエリキャッシュがサポートするクエリは以下の条件を満たす必要があります：

- クエリの実行エンジンはパイプラインであること。

  > **注意**
  >
  > パイプライン以外の実行エンジンはクエリキャッシュをサポートしていません。

- クエリのテーブルはネイティブOLAPテーブル（2.5バージョン以降サポート）またはストアと計算を分離したテーブル（3.0バージョン以降サポート）であること。外部テーブルのクエリはサポートされていません。クエリプランで実際にアクセスされるのは同期物理ビューの場合、クエリキャッシュも有効になります。非同期物理ビューは現在サポートされていません。

- クエリは単一テーブルの集計クエリまたは複数テーブルJOINの集計クエリであること。

  > **注意**
  >
  > - クエリキャッシュはブロードキャストジョインとバケットシャッフルジョインをサポートしています。
  > - クエリキャッシュはJOIN演算子を含む2つのツリー形式をサポートしています：集計後にJOINする場合とJOIN後に集計する場合。集計後にJOINするツリー形式ではシャッフルジョインはサポートされません。JOIN後に集計するツリー形式ではハッシュジョインはサポートされません。

- クエリには `rand` 、`random`、`uuid`、`sleep` などの不確定性（Nondeterminstic）関数が含まれていないこと。

クエリキャッシュはUnpartitioned、Multi-Column Partitioned、Single-Column Partitionedなど、すべてのデータパーティション戦略をサポートしています。

## 製品の制約

- クエリキャッシュはパイプライン実行エンジンのPer-Tablet計算に依存しています。Per-Tablet計算は、パイプラインドライバがタブレット単位で全体のタブレットを処理することを意味します。つまり、単一のBEがアクセスするタブレットの数が実際に呼び出されるパイプラインドライバの数（つまり、実際の並行度）以上の場合、クエリキャッシュが有効になります。単一のBEがアクセスするタブレットの数がパイプラインドライバの数よりも少ない場合、各パイプラインドライバは特定のタブレットの一部のデータのみを処理し、Per-Tabletの計算結果を形成することができません。この場合、クエリキャッシュは使用されません。
- StarRocksでは、集計クエリには少なくとも4つの集計段階が含まれています。1段階の集計では、OlapScanNodeとAggregateNodeが同じフラグメントにある場合にのみ、AggregateNodeが生成するPer-Tabletの計算結果がキャッシュされます。他の段階の集計では、AggregateNodeが生成するPer-Tabletの計算結果はキャッシュされません。一部のDISTINCT集計クエリは、セッション変数 `cbo_cte_reuse` が `true` の場合に影響を受けます。データを生成するOlapScanNodeとデータを消費する1段階目のAggregateNodeが異なるフラグメントにあり、その間にExchangeNodeを介してデータが転送される場合、クエリキャッシュは使用されません。次の2つのシナリオでは、CTE最適化が採用され、クエリキャッシュは使用されません：
  - クエリの出力列に `avg(distinct)` 集計関数が含まれている場合。
  - クエリの出力列に複数のDISTINCT集計関数が含まれている場合。
- データの集計前にシャッフル操作が行われた場合、クエリキャッシュはそのデータのクエリを高速化することはできません。
- テーブルのグループ化列または重複排除列が高基数列（High-Cardinality Column）である場合、そのテーブルの集計クエリの結果は非常に大きくなります。このようなクエリはクエリキャッシュをバイパスします。
- クエリキャッシュはBEのわずかなメモリを占有し、デフォルトのキャッシュサイズは512 MBです。したがって、大きなデータ項目をキャッシュすることは適していません。また、クエリキャッシュを有効にした場合、キャッシュのヒット率が低い場合はパフォーマンスのペナルティが発生します。したがって、クエリの計算中にあるタブレットの計算結果のサイズが `query_cache_entry_max_bytes` または `query_cache_entry_max_rows` パラメータで指定された閾値を超える場合、そのクエリの後続の計算はクエリキャッシュを使用せず、代わりにパススルーメカニズムをトリガーします。

## 原理の紹介

クエリキャッシュを有効にすると、BEはクエリのローカル集計を次の2つのステージに分割します：

1. テーブルごとの集計

   BEはクエリに関連する各タブレットを個別に処理します。特定のタブレットを処理する際、BEはまずクエリキャッシュにそのタブレットの中間計算結果が存在するかどうかをチェックします。存在する場合（キャッシュヒット）、BEはその結果を直接使用します。存在しない場合（キャッシュミス）、BEはディスクからそのタブレットのデータを読み込み、ローカル集計を行い、集計結果をクエリキャッシュに格納して、後続の類似クエリで使用できるようにします。

2. タブレット間の集計

   BEはクエリに関連するすべてのタブレットの中間計算結果を収集し、これらの結果を最終結果にマージします。

   ![クエリキャッシュ - 仕組み - 1](../assets/query_cache_principle-1.png)

後続の類似クエリは、以前にキャッシュされたクエリ結果を再利用することができます。下の図に示すようなクエリでは、合計3つのタブレット（番号0から2）が関与しており、クエリキャッシュには最初のタブレット（タブレット0）の中間結果がキャッシュされています。この場合、BEはクエリキャッシュから直接タブレット0の中間計算結果を取得し、ディスク上のデータにアクセスする必要はありません。クエリキャッシュが完全にプリヒートされている場合、すべての3つのタブレットの中間計算結果が含まれており、BEはディスク上のデータにアクセスする必要はありません。

![クエリキャッシュ - 仕組み - 2](../assets/query_cache_principle-2.png)

余分なメモリを解放するために、クエリキャッシュは「最近最も使用されていない」（Least Recently Used、LRU）アルゴリズムに基づいたエントリのキャッシュアイテムの管理を行います。クエリキャッシュが設定されたキャッシュサイズの `query_cache_capacity` パラメータを超えるメモリを占有する場合、最近最も使用されていないキャッシュアイテムがクエリキャッシュから削除されます。

> **注意**
>
> 将来的には、クエリキャッシュは「Time to Live（TTL）」ベースのエントリのキャッシュアイテムの管理もサポートする予定です。

FEは、各クエリがクエリキャッシュを使用して高速化する必要があるかどうかを判断し、クエリを規格化して、クエリの意味に影響を与えない微細なテキストの詳細を除去します。

クエリキャッシュを有効にすることによるパフォーマンスの低下を防ぐために、BEは実行時に自動的にクエリキャッシュをバイパスする自己適応戦略を採用します。

## クエリキャッシュの有効化

このセクションでは、クエリキャッシュを有効にし、設定するためのパラメータとセッション変数について説明します。

### FEセッション変数

| **変数**                    | **デフォルト値** | **動的変更可能** | **説明**                                                     |
| --------------------------- | ---------- | -------------------- | ------------------------------------------------------------ |
| enable_query_cache          | false      | はい                   | クエリキャッシュを有効にするかどうかを指定します。有効な値の範囲は `true` と `false` です。`true` は有効、`false` は無効を意味します。この機能を有効にすると、クエリがこのドキュメントの「[適用シナリオ](../using_starrocks/query_cache.md#適用シナリオ)」セクションで説明されている条件を満たす場合にのみ、クエリキャッシュが有効になります。 |
| query_cache_entry_max_bytes | 4194304    | はい                   | パススルーモードをトリガーする閾値を指定します。範囲は `0` 〜 `9223372036854775807` です。1つのタブレットで生成される計算結果のバイト数または行数が `query_cache_entry_max_bytes` または `query_cache_entry_max_rows` で指定された閾値を超える場合、クエリはパススルーモードで実行されます。<br />`query_cache_entry_max_bytes` または `query_cache_entry_max_rows` の値が `0` の場合、タブレットが結果を生成しない場合でもパススルーモードが使用されます。 |
| query_cache_entry_max_rows  | 409600     | はい                   | 上記と同様です。                                                           |

### BE設定項目

以下のパラメータをBEの設定ファイル **be.conf** に設定する必要があります。パラメータの設定を変更した後は、BEを再起動する必要があります。

| **パラメータ**             | **必須** | **説明**                                                     |
| -------------------- | -------- | ------------------------------------------------------------ |
| query_cache_capacity | いいえ       | クエリキャッシュのサイズを指定します。単位はバイトです。デフォルトは512 MBです。<br />各BEには独自のクエリキャッシュがあり、自身のクエリキャッシュのみがポピュレート（Populate）およびプローブ（Probe）されます。クエリキャッシュはBEのメモリを占有します。<br />クエリキャッシュのサイズは4 MB未満にすることはできません。現在のBEのメモリ容量が期待するクエリキャッシュのサイズを満たせない場合は、BEのメモリ容量を増やしてから適切なクエリキャッシュのサイズを設定してください。 |

## 最大のキャッシュヒット率を確保する

後続のクエリが以前にキャッシュされたクエリ結果と完全に等価ではない場合でも、クエリキャッシュはキャッシュヒット率を効果的に向上させることができます。主なクエリシナリオは次のとおりです：

- 意味的に等価なクエリ
- パーティションの重複スキャンクエリ
- 追加の書き込みのみを含む（UPDATEまたはDELETE操作なし）データのクエリ

### セマンティックに等価なクエリ

二つのクエリが類似している場合（これらのクエリが文字通り完全に一致している必要はなく、その実行計画がセマンティックに等しい場合）、これらのクエリは互いの以前に計算された結果を再利用することができます。一般的に言えば、セマンティック等価とは、二つのクエリが同じデータソースから計算され、同じ計算方法を使用し、類似した実行計画を持っていることを指します。厳密に言えば、二つのクエリがセマンティックに等価であるかどうかを判断するルールは以下の通りです：

- 二つのクエリが複数の集約を含む場合、最初の集約がセマンティックに等価であれば、クエリはセマンティックに等価と判断されます。例えば、以下の二つのクエリ、Q1 と Q2 です。Q1 の最初の集約と Q2 の最初の集約が等価であるため、これら二つのクエリの計算結果は互いに再利用可能です。

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
                ts BETWEEN '2022-01-03 00:00:00'
                AND '2022-01-03 23:59:59'
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
        ts BETWEEN '2022-01-03 00:00:00'
        AND '2022-01-03 23:59:59'
    GROUP BY
        date_trunc('hour', ts),
        k0
    ```

- 二つのクエリが以下の四つのタイプのクエリの同じタイプに該当する場合、セマンティックに等価と判断できます。HAVING 句を含むクエリと含まないクエリは等価ではありません。同じタイプのクエリであっても、ORDER BY 句や LIMIT 句の有無は、二つのクエリがセマンティックに等価であるかどうかに影響しません。

  - GROUP BY による集約

    ```SQL
    SELECT <GroupByItems>, <AggFunctionItems> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>] 
    ```

    > **注釈**
    >
    > HAVING 句はオプションです。

  - GROUP BY DISTINCT による集約

    ```SQL
    SELECT DISTINCT <GroupByItems>, <Items> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    HAVING <HavingPredicate>
    ```

    > **注釈**
    >
    > HAVING 句はオプションです。

  - GROUP BY を使用しない集約

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

  - GROUP BY DISTINCT を使用しない集約

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

- 二つのクエリのいずれかが `PartitionColumnRangePredicate` を含む場合、`PartitionColumnRangePredicate` を削除してからセマンティック等価かどうかを判断します。`PartitionColumnRangePredicate` は、列がパーティション列であり、以下の五つのタイプのいずれかである述語を指します：

  - `col between v1 and v2`：パーティション列の値が区間 [v1, v2] に属しており、`v1`、`v2` は定数式です。
  - `v1 < col and col < v2`：パーティション列の値が区間 (v1, v2) に属しており、`v1`、`v2` は定数式です。
  - `v1 < col and col <= v2`：パーティション列の値が区間 (v1, v2] に属しており、`v1`、`v2` は定数式です。
  - `v1 <= col and col < v2`：パーティション列の値が区間 [v1, v2) に属しており、`v1`、`v2` は定数式です。
  - `v1 <= col and col <= v2`：パーティション列の値が区間 [v1, v2] に属しており、`v1`、`v2` は定数式です。

- 二つのクエリの SELECT 出力列を並び替えた後が同じであれば、セマンティックに等価と判断されます。

- 二つのクエリが共に GROUP BY 句を含み、それらの GROUP BY 出力列を並び替えた後が同じであれば、セマンティックに等価と判断されます。

- 二つのクエリが共に WHERE 句を含み、それらの WHERE 句から `PartitionColumnRangePredicate` を除いた残りの述語が完全に等価であれば、セマンティックに等価と判断されます。

- 二つのクエリが共に HAVING 句を含み、それらの HAVING 句の述語が完全に等価であれば、セマンティックに等価と判断されます。

例えば、以下の標準テーブル `lineorder_flat` を例にとります：

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
PARTITION p7 VALUES [('1998-01-01'), ('1999-01-01']))
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

以下の二つのクエリ Q1 と Q2 は、次の処理を行った後、セマンティックに等価と判断できます：

1. SELECT 出力列の並び替え。
2. GROUP BY 出力列の並び替え。
3. ORDER BY 出力列の削除。
4. WHERE 句内の述語の並び替え。
5. `PartitionColumnRangePredicate` の追加。

- Q1

  ```SQL
  SELECT sum(lo_revenue), year(lo_orderdate) AS year, p_brand
  FROM lineorder_flat
  WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
  GROUP BY year, p_brand
  ORDER BY year, p_brand;
  ```

- Q2

  ```SQL
  SELECT year(lo_orderdate) AS year, p_brand, sum(lo_revenue)
  FROM lineorder_flat
  WHERE s_region = 'AMERICA' AND p_category = 'MFGR#12' AND 
     lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
  GROUP BY p_brand, year(lo_orderdate)
  ```

二つのクエリが等価であるかどうかの判断は、クエリの物理的な計画に基づいているため、クエリの文字通りの違いはセマンティック等価の判断に影響しません。さらに、クエリ内で定数式の計算を消去でき、`cast` 式はクエリの計画段階で既に消去されているため、これらの式はセマンティック等価の判断に影響しません。また、Column と Relation のエイリアスも等価判断には影響しません。

### パーティションが重なるクエリのスキャン

Query Cache は述語の分解をサポートしています。

述語の分解により、部分的な計算結果の再利用が可能になります。クエリにパーティション述語（つまり、パーティション列を含む述語）が含まれ、パーティション述語が範囲を示している場合、その範囲をデータテーブルのパーティションに従って小さな区間に分解することができます。各区間内の計算結果は、他のクエリで再利用することができます。

以下のデータテーブル `t0` を例にとります：

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


表 `t0` は日ごとにパーティション分けされており、`ts` がパーティション列です。以下の4つのクエリでは、Q2、Q3、Q4がQ1の一部の計算結果を再利用できます:

- Q1

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1のパーティション述語 `ts between '2022-01-02 12:30:00' and '2022-01-14 23:59:59'` は以下のようにいくつかの区間に分解できます:

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
  WHERE ts >= '2022-01-02 12:30:00' AND ts < '2022-01-05 00:00:00'
  GROUP BY day;
  ```

  Q2はQ1の以下の区間の計算結果を再利用できます:

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ```

- Q3

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-01 12:30:00' AND ts <= '2022-01-10 12:00:00'
  GROUP BY day;
  ```

  Q3はQ1の以下の区間の計算結果を再利用できます:

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

  Q4はQ1の以下の区間の計算結果を再利用できます:

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

部分的な結果の再利用のサポートはパーティション戦略に依存しており、以下の表で説明されています。

| **パーティション戦略**              | **部分的な結果の再利用のサポート**        |
| ------------------------- | ------------------------------- |
| Unpartitioned             | サポートされていません                          |
| Multi-Column Partitioned  | サポートされていません<br />**注記**<br />将来的にサポートされる可能性があります。 |
| Single-Column Partitioned | サポートされています                            |

### 追加書き込みのみを行うクエリ

Query CacheはマルチバージョンのCacheメカニズムをサポートしています。

データがインポートされると、Tabletは新しいバージョンを生成し、それによりQuery Cache内のキャッシュされた結果のTabletバージョンが実際のTabletバージョンよりも古くなります。この場合、マルチバージョンのCacheメカニズムはQuery Cache内のキャッシュされた結果とディスク上に保存された増分データをマージしようとし、新しいクエリが最新バージョンのTabletデータを取得できるようにします。マルチバージョンのCacheメカニズムの動作は、データモデル、クエリタイプ、およびデータ更新タイプによって制限されます。

異なるデータモデルとクエリタイプに対するマルチバージョンのCacheメカニズムのサポートは以下の表で説明されています。

| **データモデル** | **クエリタイプ**               | **マルチバージョン Cache メカニズムのサポート**                                  |
| ------------ | -------------------------- | ------------------------------------------------------------ |
| 明細モデル     | <ul><li>ベーステーブルクエリ</li><li>同期物化ビュークエリ</li></ul>   | <ul><li>ベーステーブルクエリ：増分バージョンに削除レコードが含まれている場合のみサポートされません。それ以外の場合はサポートされます。</li><li>同期物化ビュークエリ：GROUP BY、HAVING、またはWHERE句で集約列を参照している場合のみサポートされません。それ以外の場合はサポートされます。</li></ul> |
| 集約モデル     | ベーステーブルクエリまたは同期物化ビュークエリ | 以下のシナリオではサポートされません：ベーステーブルのSchemaに集約関数 `replace` が含まれている。GROUP BY、HAVING、またはWHERE句で集約列を参照している。増分バージョンに削除レコードが含まれている。それ以外の場合はサポートされます。 |
| 更新モデル     | 該当なし                     | Query Cacheはサポートされていますが、マルチバージョンのCacheメカニズムはサポートされていません。                |
| 主キーモデル     | 該当なし                     | Query Cacheはサポートされていますが、マルチバージョンのCacheメカニズムはサポートされていません。                |

異なるデータ更新タイプがマルチバージョンのCacheメカニズムに与える影響は以下の通りです:

- データ削除

  Tabletの増分データを削除すると、マルチバージョンのCacheメカニズムが無効になります。

- データインポート

  - Tabletに空の新バージョンが生成された場合、Query Cache内の既存のデータは引き続き有効であり、クエリ時にはQuery Cacheから直接データを取得できます。
  - Tabletに非空の新バージョンが生成された場合、Query Cache内の既存のデータは引き続き有効ですが、既存のデータのバージョンは最新のTabletバージョンよりも古くなっています。この場合、既存のデータのバージョンからTabletの最新バージョンまでの増分データをTabletから読み取り、既存のデータと増分バージョンの計算結果をマージし、その後マージされたデータ結果をQuery Cacheに再度充填する必要があります。

- テーブル構造 (Schema) の変更とTabletのトリミング

  テーブル構造の変更とTabletのトリミングは新しいTabletを生成し、Query Cache内の既存のデータを無効にします。

## 監視指標

Query Cacheを使用するクエリでは、Profileに `CacheOperator` の統計情報が表示されます。以下の図のようになります。

![Query Cache - Metrics Overview](../assets/query-cache-metrics-overview.png)

まず、元の実行計画に含まれる `OlapScanOperator` のPipelineでは、`OlapScanOperator` の後続オペレーターから集約オペレーターまでのオペレーター名に `ML_` のプレフィックスが追加され、これは現在のPipelineが `MultilaneOperator` を導入してPer-Tablet計算を行っていることを示します。`ML_CONJUGATE_AGGREGATE` オペレーターの上に `CacheOperator` が挿入され、この `CacheOperator` はPassthrough、Populate、Probeの3つのモードでのQuery Cacheの動作ロジックを処理します。`CacheOperator` のProfileには、Query Cacheの使用状況を統計する以下の指標があります。

| **指標**                  | **説明**                                                 |
| ------------------------- | -------------------------------------------------------- |
| CachePassthroughBytes     | Passthroughモードで生成されたバイト数。                      |
| CachePassthroughChunkNum  | Passthroughモードで生成されたChunk数。                   |
| CachePassthroughRowNum    | Passthroughモードで生成された行数。                        |
| CachePassthroughTabletNum | Passthroughモードで計算されたTablet数。                  |
| CachePassthroughTime      | Passthroughモードでの計算時間。                           |
| CachePopulateBytes        | Populateモードで生成されたバイト数。                         |
| CachePopulateChunkNum     | Populateモードで生成されたChunk数。                      |
| CachePopulateRowNum       | Populateモードで生成された行数。                           |
| CachePopulateTabletNum    | Populateモードで計算されたTablet数。                     |
| CachePopulateTime         | Populateモードでの計算時間。                              |
| CacheProbeBytes           | Probeモードでキャッシュヒットした際に生成されたバイト数。 |
| CacheProbeChunkNum        | Probeモードでキャッシュヒットした際に生成されたChunk数。   |
| CacheProbeRowNum          | Probeモードでキャッシュヒットした際に生成された行数。     |
| CacheProbeTabletNum       | ProbeモードでキャッシュヒットしたTablet数。               |
| CacheProbeTime            | Probeモードでの計算時間。                                 |

`CachePopulate`*`XXX`* 指標は、キャッシュミスが発生し、Query Cacheが更新された統計情報を表します。

`CachePassthrough`*`XXX`* 指標は、キャッシュミスが発生したものの、生成されたPer-Tablet計算結果が大きすぎてQuery Cacheが更新されなかった統計情報を表します。

`CacheProbe`*`XXX`* 指標は、キャッシュヒットの統計情報を表します。

マルチバージョンのCacheメカニズムでは、CachePopulateとCacheProbeの統計には重複するTabletが含まれることがあります。CachePassthroughとCacheProbeにも重複するTabletが含まれることがあります。例えば、各Tabletの結果を計算する際に、キャッシュされたTabletの歴史バージョンの計算結果がヒットし、増分バージョンを読み取って計算した後、既存のキャッシュ内容とマージされます。マージされた計算結果が `query_cache_entry_max_bytes` または `query_cache_entry_max_rows` パラメータで指定された閾値を超えない場合は、CachePopulateの統計に含まれます。それを超える場合は、CachePassthroughの統計に含まれます。

## RESTful API 操作インターフェース

- `metrics |grep query_cache`

  Query Cacheに関連する指標を表示するために使用されます。以下のように表示されます:

  ```Apache
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

  Query Cache の使用状況を表示するために使用されます。以下のように表示されます:

  ```Bash
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

  Query Cache をクリアするために使用されます。以下のように表示されます:

  ```Bash
  curl  -XPUT http://<be_host>:<be_http_port>/api/query_cache/invalidate_all
    
  {
      "status": "OK"
  }
  ```

パラメータの説明は以下の通りです：

- `be_host`：BEノードのIPアドレス。
- `be_http_port`：BEノードのHTTPポート番号。

## 注意事項

- 一部のクエリは初めて実行される際、Query Cache を充填する必要があるため、わずかなパフォーマンスのペナルティが発生し、遅延が増加する可能性があります。
- Query Cache に大きなメモリ容量を設定すると、BEでクエリ評価に使用できるメモリ容量を占有します。Query Cache のメモリ容量は、クエリ評価に使用できるメモリ容量の1/6を超えないように設定することをお勧めします。
- `pipeline_dop` の値より少ない Tablet 数を Pipeline が処理する場合、Query Cache は有効になりません。この場合、`pipeline_dop` を小さな値に設定することができます。例えば `1` に設定します。バージョン3.0からは、クエリの並行度に応じて `pipeline_dop` を自動調整する機能がサポートされています。

## 例

### データセット

1. StarRocks クラスタにログインし、対象データベースに入り、以下のコマンドを実行してテーブル `t0` を作成します:

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

2. テーブル `t0` に以下のデータを挿入します:

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

### クエリ例

このセクションに示されている Query Cache 関連の指標統計結果は、あくまで例示であり、実際の統計結果に基づいています。

#### ステージ1のローカルアグリゲーションで Query Cache を使用

以下の3つのシナリオが含まれます：

- クエリが単一の Tablet にのみアクセスする。
- クエリが複数のパーティションの複数の Tablet にアクセスし、データテーブルが Colocated Group を使用しており、集計計算に Shuffle が不要です。
- クエリが1つのパーティションの複数の Tablet にアクセスし、集計計算に Shuffle が不要です。

クエリ例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    k0,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    k0
```

プロファイルにおける Query Cache 関連の指標統計は以下の図に示されています。

![Query Cache - Stage 1](../assets/query_cache_stage1_agg_with_cache_zh.png)

#### ステージ1のリモートアグリゲーションで Query Cache を使用しない

強制的にステージ1のアグリゲーションを使用し、かつアグリゲーション計算に複数の Tablet を跨ぐ必要がある場合、データは先に Shuffle されてから集計されます。

クエリ例：

```SQL
SET new_planner_agg_stage = 1;

SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

#### ステージ2のローカルアグリゲーションで Query Cache を使用

以下の3つのシナリオが含まれます：

- クエリのステージ2のアグリゲーションが同じタイプのアグリゲーションであり、最初のアグリゲーションはローカルで行われ、その結果が再度グローバルアグリゲーションされます。
- SELECT DISTINCT クエリです。
- DISTINCT アグリゲーション関数 `sum(distinct)`、`count(distinct)`、または `avg(distinct)` を含むクエリです。このタイプのクエリは通常、ステージ3またはステージ4のアグリゲーションを使用しますが、`set new_planner_agg_stage = 1` を設定することでステージ2のアグリゲーションを強制的に使用することもできます。もし `avg(distinct)` の DISTINCT アグリゲーション関数を含むクエリでステージ2のアグリゲーションを使用する場合は、`set cbo_cte_reuse = false` を設定して CTE 最適化を無効にする必要があります。

クエリ例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

プロファイルにおける Query Cache 関連の指標統計は以下の図に示されています。

![Query Cache - Stage 2](../assets/query_cache_stage2_agg_with_cache_zh.png)

#### ステージ3のローカルアグリゲーションで Query Cache を使用

単一の DISTINCT アグリゲーション関数を含む GROUP BY アグリゲーションクエリです。

サポートされている DISTINCT アグリゲーション関数には `sum(distinct)`、`count(distinct)`、`avg(distinct)` があります。

> **注意**
>
> `avg(distinct)` を使用する場合は CTE 最適化を無効にする必要があります。コマンドは以下の通りです：`set cbo_cte_reuse = false`。

クエリ例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd;
```

プロファイルにおける Query Cache 関連の指標統計は以下の図に示されています。

![Query Cache - Stage 3](../assets/query_cache_stage3_agg_with_cache_zh.png)

#### ステージ4のローカルアグリゲーションで Query Cache を使用

単一の DISTINCT アグリゲーション関数を含む非 GROUP BY アグリゲーションクエリです。たとえば、典型的な重複排除クエリです。

クエリ例：

```SQL
SELECT
    count(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
```

プロファイルにおける Query Cache 関連の指標統計は以下の図に示されています。

![Query Cache - Stage 4](../assets/query_cache_stage4_agg_with_cache_zh.png)

#### 2つのクエリの最初のアグリゲーションが意味的に等価で Query Cache のキャッシュ結果を再利用

たとえば、以下の2つのクエリ、Q1 と Q2。Q1 と Q2 は複数のアグリゲーションを含んでいますが、それらの最初のアグリゲーションは意味的に等価であり、したがって、互いの Query Cache にキャッシュされた計算結果を再利用することができると判断されます。

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
              ts BETWEEN '2022-01-03 00:00:00'
              AND '2022-01-03 23:59:59'
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
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59'
  GROUP BY
      date_trunc('hour', ts),
      k0
  ```

Q1 のクエリでの CachePopulate カテゴリの指標統計結果は以下の図に示されています。

![Query Cache - Q1 Metrics](../assets/query_cache_reuse_Q1_zh.png)

Q2 のクエリでの CacheProbe カテゴリの指標統計結果は以下の図に示されています。

![Query Cache - Q2 Metrics](../assets/query_cache_reuse_Q2_zh.png)

#### CTE 最適化を使用した DISTINCT クエリで Query Cache を使用しない

`set cbo_cte_reuse = true` を設定して CTE 最適化を有効にした後、DISTINCT アグリゲーション関数を含むいくつかのシナリオでは、計算結果はキャッシュされません。以下に例をいくつか挙げます：

- DISTINCT アグリゲーション関数 `avg(distinct)` を含むクエリ。

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0
  FROM
      t0
  WHERE
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 1](../assets/query_cache_distinct_with_cte_Q1_zh.png)

- 同じ列に対する複数の DISTINCT アグリゲーション関数を含むクエリ。

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0,
      sum(distinct v1) AS __c_1,
      count(distinct v1) AS __c_2
  FROM
      t0
  WHERE
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 2](../assets/query_cache_distinct_with_cte_Q2_zh.png)

- 異なる列に対する複数の DISTINCT アグリゲーション関数を含むクエリ。

  ```SQL
  SELECT
      sum(distinct v1) AS __c_1,
      count(distinct v0) AS __c_2
  FROM
      t0
  WHERE
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_zh.png)

## ベストプラクティス


テーブル作成時には、合理的なパーティション戦略を設定し、適切なデータ分布方法を選択します。これには以下が含まれます：

- DATE型のデータのみを含む列をパーティション列として選択します。複数のDATE型データを含む列がテーブルに存在する場合は、以下の2つの条件を満たす列をパーティション列として選択してください：
  - 列の値はデータの挿入と共に前方へと進行します。
  - その列はクエリの時間範囲を定義するために使用されます。
- 適切なパーティションの幅を選択します。最新のデータ挿入により、テーブルの最新パーティションが変更される可能性があり、最新パーティションに関連するキャッシュエントリが不安定になり、失効しやすくなります。
- バケット数が数十個程度であることを確認します。バケット数が少なすぎると、BEが処理する必要があるTabletの数が`pipeline_dop`パラメータの値よりも少ない場合、Query Cacheは機能しません。

---
displayed_sidebar: "Japanese"
---

# クエリプロファイルの分析

このトピックでは、クエリプロファイルの確認方法について説明します。 クエリプロファイルは、クエリに関与するすべてのワーカーノードの実行情報を記録します。 クエリプロファイルを使用すると、StarRocksクラスターのクエリのパフォーマンスに影響を与えるボトルネックを迅速に特定できます。

## クエリプロファイルを有効にする

v2.5より前のStarRocksバージョンでは、変数`is_report_success` を `true` に設定することでクエリプロファイルを有効にできます。

```SQL
SET is_report_success = true;
```

v2.5以降のStarRocksバージョンでは、変数`enable_profile` を `true` に設定することでクエリプロファイルを有効にできます。

```SQL
SET enable_profile = true;
```

### ランタイムプロファイル

v3.1以降、StarRocksはランタイムプロファイル機能をサポートし、クエリが完了する前にクエリプロファイルにアクセスできるようになりました。

この機能を使用するには、`enable_profile` を `true` に設定するだけでなく、セッション変数`runtime_profile_report_interval` を設定する必要があります。 プロファイル報告間隔を指定する `runtime_profile_report_interval` (単位: 秒、デフォルト: `10`) は、デフォルトで10秒に設定されており、クエリが10秒を超えると、自動的にランタイムプロファイル機能が有効になります。

```SQL
SET runtime_profile_report_interval = 10;
```

ランタイムプロファイルは、通常のクエリプロファイルと同じ情報を表示します。 実行中のクエリのパフォーマンスに関する貴重な洞察を得るために、通常のクエリプロファイルと同様に分析できます。

ただし、ランタイムプロファイルは不完全な場合があります。 実行計画の一部のオペレータは他のオペレータに依存する場合があるためです。 実行中のオペレータと完了したオペレータを簡単に区別するには、「ステータス: 実行中」とマークされた実行中のオペレータがあります。

## クエリプロファイルへのアクセス

> **注意**
>
> StarRocksのEnterprise Editionをご利用の場合は、StarRocks Managerを使用してクエリプロファイルにアクセスして表示できます。

StarRocksのCommunity Editionをご利用の場合は、次の手順に従ってクエリプロファイルにアクセスできます。

1. ブラウザで `http://<fe_ip>:<fe_http_port>` を入力します。
2. 表示されたページで、上部ナビゲーションペインの **queries** をクリックします。
3. **終了したクエリ** リストで、確認したいクエリを選択し、 **Profile** 列のリンクをクリックします。

![img](../assets/profile-1.png)

ブラウザは、対応するクエリプロファイルが含まれる新しいページにリダイレクトします。

![img](../assets/profile-2.png)

## クエリプロファイルの解釈

### クエリプロファイルの構造

以下はクエリプロファイルの例です。

```SQL
Query:
  概要:
  プランナー:
  Execution Profile 7de16a85-761c-11ed-917d-00163e14d435:
    Fragment 0:
      Pipeline (id=2):
        EXCHANGE_SINK (plan_node_id=18):
        LOCAL_MERGE_SOURCE (plan_node_id=17):
      Pipeline (id=1):
        LOCAL_SORT_SINK (plan_node_id=17):
        AGGREGATE_BLOCKING_SOURCE (plan_node_id=16):
      Pipeline (id=0):
        AGGREGATE_BLOCKING_SINK (plan_node_id=16):
        EXCHANGE_SOURCE (plan_node_id=15):
    Fragment 1:
       ...
    Fragment 2:
       ...
```

クエリプロファイルには、3つのセクションが含まれています。

- Fragment: 実行ツリー。 1つのクエリを1つ以上のフラグメントに分割できます。
- Pipeline: 実行チェーン。 実行チェーンには枝がありません。 フラグメントを複数のパイプラインに分割できます。
- Operator: パイプラインは複数のオペレータで構成されます。

![img](../assets/profile-3.png)

*フラグメントには複数のパイプラインが含まれています。*

### 主要なメトリクス

クエリプロファイルには、クエリ実行の詳細を示す多くのメトリクスが含まれています。 ほとんどの場合、オペレータの実行時間と処理されたデータのサイズだけを観察すれば十分です。 ボトルネックを見つけた後は、それらを対応する方法で解決できます。

#### 概要

| メトリック    | 説明                                                       |
| ------------ | --------------------------------------------------------- |
| Total        | プランニング、実行、プロファイリングに費やされた合計時間。 |
| QueryCpuCost | クエリの合計CPU時間コスト。                                   |
| QueryMemCost | クエリの合計メモリコスト。                                     |

#### オペレータのための普遍的なメトリクス

| メトリック            | 説明                                  |
| ----------------- | ------------------------------------ |
| OperatorTotalTime | オペレータの合計CPU時間コスト。            |
| PushRowNum        | オペレータがプッシュしたデータの総数。       |
| PullRowNum        | オペレータが取得したデータの総数。         |

#### ユニークなメトリクス

| メトリック            | 説明                    |
| ----------------- | ---------------------- |
| IOTaskExecTime    | すべてのI/Oタスクの合計実行時間。 |
| IOTaskWaitTime    | すべてのI/Oタスクの合計待機時間。 |
| MorselsCount      | I/Oタスクの総数。             |

#### スキャンオペレータ

| メトリック                          | 説明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| Table                           | テーブル名。                                                    |
| ScanTime                        | スキャンに要した合計時間。スキャンは非同期のI/Oスレッドプールで実行されます。 |
| TabletCount                     | タブレットの数。                                                 |
| PushdownPredicates              | プッシュダウンされた述語の数。                                          |
| BytesRead                       | StarRocksによって読み取られたデータのサイズ。                             |
| CompressedBytesRead             | StarRocksによって読み取られた圧縮データのサイズ。                                |
| IOTime                          | 合計I/O時間。                                                    |
| BitmapIndexFilterRows           | Bitmapインデックスによってフィルタリングされたデータの行数。                           |
| BloomFilterFilterRows           | Bloomフィルタによってフィルタリングされたデータの行数。                             |
| SegmentRuntimeZoneMapFilterRows | ランタイムZone Mapによってフィルタリングされたデータの行数。                         |
| SegmentZoneMapFilterRows        | Zone Mapによってフィルタリングされたデータの行数。                         |
| ShortKeyFilterRows              | ショートキーによってフィルタリングされたデータの行数。                           |
| ZoneMapIndexFilterRows          | Zone Mapインデックスによってフィルタリングされたデータの行数。                      |

#### Exchangeオペレータ

| メトリック            | 説明                                                   |
| ----------------- | ----------------------------------------------------- |
| PartType          | データ分配タイプ。有効な値: `UNPARTITIONED`, `RANDOM`, `HASH_PARTITIONED`, `BUCKET_SHUFFLE_HASH_PARTITIONED`。 |
| BytesSent         | 送信されたデータのサイズ。                                       |
| OverallThroughput | 全体のスループット。                                              |
| NetworkTime       | データパッケージの送信時間 (受信後の処理時間は除く)。このメトリックの計算方法と例外発生時の詳細についてはFAQを参照してください。 |
| WaitTime          | 送信側のキューがいっぱいになるための待機時間。                     |

#### Aggregateオペレータ

| メトリック             | 説明                                  |
| ------------------ | ------------------------------------ |
| GroupingKeys       | グループ化キーの名前 (GROUP BY列)。          |
| AggregateFunctions | 集約関数。                                |
| AggComputeTime     | 集約関数によって消費される計算時間。          |
| ExprComputeTime    | 式によって消費される計算時間。               |
| HashTableSize      | ハッシュテーブルのサイズ。                    |

#### Joinオペレータ

| メトリック                    | 説明              |
| ------------------------- | ---------------- |
| JoinPredicates            | JOIN操作の述語。       |
| JoinType                  | JOINタイプ。         |
| BuildBuckets              | ハッシュテーブルのバケット数。 |
| BuildHashTableTime        | ハッシュテーブルの構築に使用される時間。 |
| ProbeConjunctEvaluateTime | Probe Conjunctによって消費される時間。 |
| SearchHashTableTimer      | ハッシュテーブルの検索に使用される時間。 |

#### Window Functionオペレータ

| メトリック             | 説明                                                             |
| ------------------ | ------------------------------------------------------------------- |
| ComputeTime        | ウィンドウ関数によって消費される計算時間。                                              |
| PartitionKeys      | パーティションキーの名前 (PARTITION BY列)。                                  |
| AggregateFunctions | 集約関数。                                                         |

#### Sortオペレータ

| メトリック | 説明                                                   |
| ------ | ----------------------------------------------------- |
| SortKeys | ソートキーの名前 (ORDER BY列)。                                 |
| SortType | 結果のソートタイプ: すべての結果をリストアップするか、トップnの結果をリストアップするか。 |

#### TableFunctionオペレータ

| メトリック                | 説明                                     |
| ------------------------- | --------------------------------------- |
| TableFunctionExecTime     | テーブル関数によって消費される計算時間。             |
| TableFunctionExecCount    | テーブル関数が実行された回数。                       |

#### Projectオペレータ

| メトリック                   | 説明                               |
| -------------------------- | --------------------------------- |
| ExprComputeTime            | 式によって消費される計算時間。             |
| CommonSubExprComputeTime   | 共通部分式によって消費される計算時間。          |

#### LocalExchangeオペレータ

| メトリック | 説明                                                   |
| ------ | ----------------------------------------------------- |
| Type   | ローカルエクスチェンジの種類。有効な値: `Passthrough`, `Partition`, `Broadcast`。 |
| ShuffleNum | シャッフルの回数。このメトリックは、`Type` が `Partition` の場合のみ有効です。       |

#### Hive Connector

| メトリック              | 説明                                   |
| ----------------------- | ------------------------------------- |
| ScanRanges             | スキャンされるタブレットの数。                  |
| ReaderInit             | リーダーの初期化時間。                        |
| カラムReadTime | リーダーがデータを読み取って解析するのにかかる時間。 |
| ExprFilterTime | 式をフィルタリングする時間。                    |
| RowsRead | 読み込まれたデータ行の数。                     |

#### インプットストリーム

| メトリック | 説明                                                     |
| ----------- | -------------------------------------------------------- |
| AppIOBytesRead | アプリケーションレイヤーからI/Oタスクによって読み込まれたデータのサイズ。 |
| AppIOCounter | アプリケーションレイヤーからのI/Oタスクの数。                              |
| AppIOTime | アプリケーションレイヤーからのI/Oタスクによってデータを読み込むのにかかる合計時間。|
| FSBytesRead | ストレージシステムによって読み込まれたデータのサイズ。                    |
| FSIOCounter | ストレージレイヤーからのI/Oタスクの数。                              |
| FSIOTime | ストレージレイヤーによるデータ読み込みにかかる合計時間。                    |

### オペレーターによる時間の消費

- OlapScanおよびConnectorScanオペレーターの時間消費は、`OperatorTotalTime + ScanTime`に等しい。スキャンオペレーターは非同期I/OスレッドプールでI/O操作を行うため、ScanTimeは非同期I/O時間を表す。
- Exchangeオペレーターの時間消費は、`OperatorTotalTime + NetworkTime`に等しい。ExchangeオペレーターはbRPCスレッドプールでデータパッケージの送受信を行うため、NetworkTimeはネットワーク転送にかかる時間を表す。
- その他のオペレーターの時間コストは`OperatorTotalTime`である。

### メトリックのマージおよびMIN/MAX

パイプラインエンジンは並列計算エンジンである。各フラグメントは複数のマシンに分散して並列処理され、各マシン上のパイプラインは複数の同時実行インスタンスとして並行して実行される。そのため、StarRocksは同じメトリックをマージし、すべての同時実行インスタンスの各メトリックの最小値と最大値を記録する。

異なるタイプのメトリックに対して異なるマージ戦略が採用されている。

- 時間メトリックは平均値である。例えば:
  - `OperatorTotalTime`はすべての同時実行インスタンスの平均時間コストを表す。
  - `__MAX_OF_OperatorTotalTime`はすべての同時実行インスタンスの間で最大の時間コストを表す。
  - `__MIN_OF_OperatorTotalTime`はすべての同時実行インスタンスの間で最小の時間コストを表す。

```SQL
             - OperatorTotalTime: 2.192us
               - __MAX_OF_OperatorTotalTime: 2.502us
               - __MIN_OF_OperatorTotalTime: 1.882us
```

- 非時間メトリックは合計値である。例えば:
  - `PullChunkNum`はすべての同時実行インスタンスの合計数を表す。
  - `__MAX_OF_PullChunkNum`はすべての同時実行インスタンスの中で最大の値を表す。
  - `__MIN_OF_PullChunkNum`はすべての同時実行インスタンスの中で最小の値を表す。

  ```SQL
                 - PullChunkNum: 146.66K (146660)
                   - __MAX_OF_PullChunkNum: 24.45K (24450)
                   - __MIN_OF_PullChunkNum: 24.435K (24435)
  ```

- 最小値と最大値を持たない特殊なメトリックがいくつかあり、それらはすべての同時実行インスタンスの間で同一の値を持つ（例: `DegreeOfParallelism`）。

#### MINとMAXの間の大きな違い

通常、MINとMAXの値の大きな違いは、データが不均衡であることを示す。これは集計やJOIN操作中に起こりうる可能性がある。

```SQL
             - OperatorTotalTime: 2m48s
               - __MAX_OF_OperatorTotalTime: 10m30s
               - __MIN_OF_OperatorTotalTime: 279.170us
```

## テキストベースのプロファイル解析を実行する

v3.1以降、StarRocksはより使いやすいテキストベースのプロファイル解析機能を提供しています。この機能を使用すると、クエリの最適化のためのボトルネックや機会を効率的に特定できます。

### 既存のクエリを解析する

クエリのプロファイルを、クエリが実行中であるか完了しているかに関わらず、その`QueryID`を使用して解析することができます。

#### プロファイルをリストする

次のSQLステートメントを実行して、既存のプロファイルをリストします。

```sql
SHOW PROFILELIST;
```

例:

```sql
MySQL > show profilelist;
+--------------------------------------+---------------------+-------+----------+---------------------------------------------+
| QueryId                              | StartTime           | Time  | State    | Statement                                   |
+--------------------------------------+---------------------+-------+----------+---------------------------------------------+
| b8289ffc-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:27 | 86ms  | Finished | SELECT o_orderpriority, COUNT(*) AS order_count\nFROM orders\nWHERE o_orderdate >= DATE '1993-07-01'\n    AND o_orderdate < DAT ...  |
| b5be2fa8-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:23 | 67ms  | Finished | SELECT COUNT(*)\nFROM (\n    SELECT l_orderkey, SUM(l_extendedprice * (1 - l_discount)) AS revenue\n        , o_orderdate, o_sh ...  |
| b36ac9c6-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:19 | 320ms | Finished | SELECT COUNT(*)\nFROM (\n    SELECT s_acctbal, s_name, n_name, p_partkey, p_mfgr\n        , s_address, s_phone, s_comment\n    F ... |
| b037b245-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:14 | 175ms | Finished | SELECT l_returnflag, l_linestatus, SUM(l_quantity) AS sum_qty\n    , SUM(l_extendedprice) AS sum_base_price\n    , SUM(l_exten ...   |
| a9543cf4-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:02 | 40ms  | Finished | select count(*) from lineitem                |
+--------------------------------------+---------------------+-------+----------+---------------------------------------------+
5 rows in set
Time: 0.006s


MySQL > show profilelist limit 1;
+--------------------------------------+---------------------+------+----------+---------------------------------------------+
| QueryId                              | StartTime           | Time | State    | Statement                                   |
+--------------------------------------+---------------------+------+----------+---------------------------------------------+
| b8289ffc-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:27 | 86ms | Finished | SELECT o_orderpriority, COUNT(*) AS order_count\nFROM orders\nWHERE o_orderdate >= DATE '1993-07-01'\n    AND o_orderdate < DAT ... |
+--------------------------------------+---------------------+------+----------+---------------------------------------------+
1 row in set
Time: 0.005s
```

このSQLステートメントを使用すると、各クエリに関連付けられた`QueryId`を簡単に取得できます。`QueryId`は、さらなるプロファイル解析および調査において重要な識別子となります。

#### プロファイルを解析する

取得した`QueryId`を使用して、ANALYZE PROFILEステートメントを実行し、特定のクエリのより詳細な解析を行うことができます。このSQLステートメントは、オペレータごとの最も重要であるメトリックを表示します。ただし、1つ以上のプランノードのIDを指定することで、対応するメトリックを表示することができます。この機能により、クエリのパフォーマンスのより深い理解と、ターゲット指向の最適化が可能となります。

例1: プランノードIDを指定せずにプロファイルを解析する:

![img](../assets/profile-16.png)

例2: プランノードIDを指定してプロファイルを解析し、対応するメトリックを表示する:

![img](../assets/profile-17.png)

ANALYZE PROFILEステートメントは、ランタイムプロファイルを分析するための改良された方法を提供します。このステートメントは、ブロックされた、実行中、完了など異なる状態のオペレータを識別します。さらに、このステートメントは、処理された行数に基づいて全体の進行状況、および個々のオペレータの進行状況を包括的に表示し、クエリの実行とパフォーマンスの深い理解を可能とします。この機能は、StarRocksのクエリのプロファイリングと最適化をさらに助けます。

例3: 実行中のクエリのランタイムプロファイルを解析する:

![img](../assets/profile-20.png)

### シミュレートされたクエリを解析する

EXPLAIN ANALYZEステートメントを使用して、与えられたクエリをシミュレートし、そのプロファイルを解析することもできます。

```sql
EXPLAIN ANALYZE <sql>
```

現在、EXPLAIN ANALYZEは、クエリ（SELECT）ステートメントとINSERT INTOステートメントの2種類のSQLステートメントをサポートしています。INSERT INTOのステートメントのプロファイルをシミュレーションして解析することができますが、実際にデータは挿入されません。プロファイル解析の過程で意図しないデータの変更がないように、デフォルトでトランザクションが中止されます。

例1: 特定のクエリのプロファイルを解析する:

![img](../assets/profile-18.png)

例2: INSERT INTO操作のプロファイルを解析する:

![img](../assets/profile-19.png)

## クエリプロファイルを可視化
StarRocksエンタープライズエディションのユーザーの場合、StarRocks Managerを使用してクエリプロファイルを視覚化できます。

**プロファイルの概要**ページには、総実行時間「ExecutionWallTime」、I/Oメトリクス、ネットワーク伝送サイズ、CPUとI/O時間の割合など、いくつかの要約メトリクスが表示されます。

![img](../assets/profile-4.jpeg)

オペレーター（ノード）のカードをクリックすると、ページの右側に詳細情報が表示されます。3つのタブがあります。

- **Node**: このオペレーターの主要メトリクス。
- **Node Detail**: このオペレーターのすべてのメトリクス。
- **Pipeline**: オペレーターが属するパイプラインのメトリクス。これはスケジューリングに関連するため、このタブには注意を払う必要はありません。

![img](../assets/profile-5.jpeg)

### ボトルネックの特定

オペレーターが使用する時間の割合が大きいほど、そのカードの色が濃くなります。これはクエリのボトルネックを簡単に特定できるようにします。

![img](../assets/profile-6.jpeg)

### データがスケーブされているかどうかを確認する

時間の大部分を取るオペレーターのカードをクリックし、「MaxTime」と「MinTime」を確認します。通常、「MaxTime」と「MinTime」の間の顕著な違いは、データがスケーブされていることを示します。

次に、**Node Detail**タブをクリックし、例えば、集約オペレーターのメトリクス「PushRowNum」がデータスケーブを示していることを確認します。

![img](../assets/profile-7.jpeg)

### パーティション分割またはバケット戦略が有効かどうかを確認する

`EXPLAIN <sql_statement>`を使用して対応するクエリプランを表示することで、パーティション分割またはバケット戦略が有効かどうかを確認できます。

![img](../assets/profile-9.png)

### 正しいマテリアライズドビューが使用されているかを確認する

対応するスキャンオペレーターをクリックし、「Node Detail」タブで「Rollup」フィールドを確認します。

![img](../assets/profile-10.jpeg)

### JOINプランが左側と右側のテーブルに適しているかどうかを確認する

通常、StarRocksはJoinの右側のテーブルとして小さいテーブルを選択します。クエリプロファイルがそれ以外を示す場合には例外が発生します。

![img](../assets/profile-11.jpeg)

### JOINの分布タイプが正しいかどうかを確認する

データ分布タイプに応じて、Exchangeオペレーターは次の3つのタイプに分類されます。

- `UNPARTITIONED`: ブロードキャスト。データはいくつかのコピーになり、複数のBEに分散されます。
- `RANDOM`: ラウンドロビン。
- `HASH_PARTITIONED`および`BUCKET_SHUFFLE_HASH_PARTITIONED`: シャッフル。`HASH_PARTITIONED`と`BUCKET_SHUFFLE_HASH_PARTITIONED`の違いは、ハッシュコードを計算するために使用されるハッシュ関数にあります。

内部結合の場合、右側のテーブルは`HASH_PARTITIONED`および`BUCKET_SHUFFLE_HASH_PARTITIONED`タイプまたは`UNPARTITIONED`タイプになります。通常、右側のテーブルに100K行未満がある場合にのみ、`UNPARTITIONED`タイプが採用されます。

次の例では、Exchangeオペレーターのタイプがブロードキャストであるにもかかわらず、オペレーターによって転送されるデータのサイズが大幅に閾値を超えています。

![img](../assets/profile-12.jpeg)

### JoinRuntimeFilterが有効になっているかを確認する

Joinの右側の子がハッシュテーブルを構築している場合、ランタイムフィルターが作成されます。このランタイムフィルターは、Scanオペレーターに可能であればプッシュダウンされるため、スキャンオペレーターの**Node Detail**タブで`JoinRuntimeFilter`関連のメトリクスを確認できます。

![img](../assets/profile-13.jpeg)

## FAQ

### Exchangeオペレーターの時間コストが異常な理由は？

![img](../assets/profile-14.jpeg)

Exchangeオペレーターの時間コストは、CPU時間とネットワーク時間の2つの部分から構成されます。ネットワーク時間はシステムクロックに依存します。ネットワーク時間は以下のように計算されます。

1. 送信者は、パッケージを送信するためのbRPCインターフェースを呼び出す前に`send_timestamp`を記録します。
2. 受信者は、bRPCインターフェースからパッケージを受け取った後に`receive_timestamp`を記録します（受信後の処理時間は除外されます）。
3. 処理が完了した後、受信者は応答を送信し、ネットワークレイテンシを計算します。パッケージの転送レイテンシは、`receive_timestamp` - `send_timestamp`に相当します。

マシン間のシステムクロックが一貫していない場合、Exchangeオペレーターの時間コストが例外になります。

### すべてのオペレーターの総時間コストがクエリの実行時間よりもかなり少ない理由は？

原因可能性：高並列シナリオでは、いくつかのパイプラインドライバーはスケジュール可能であるにもかかわらず、キューイングされたために時間内に処理されないことがあります。待機時間はオペレーターのメトリクスではなく、「PendingTime」、「ScheduleTime」、「IOTaskWaitTime」に記録されます。

例：

プロファイルから、`ExecutionWallTime`が約55ミリ秒であることがわかります。しかし、すべてのオペレーターの総時間コストは10ミリ秒未満であり、これは「ExecutionWallTime」よりもはるかに少ないです。

![img](../assets/profile-15.jpeg)
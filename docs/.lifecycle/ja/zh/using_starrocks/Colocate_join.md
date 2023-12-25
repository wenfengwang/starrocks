---
displayed_sidebar: Chinese
---

# Colocate Join

本セクションでは、Colocate Joinの使用方法について説明します。

Colocate Join機能は、分散システムがJoinデータの分布を実現する戦略の一つであり、データが複数のノードに分散されている場合のJoin操作によるデータ移動とネットワーク転送を減らし、クエリのパフォーマンスを向上させることができます。

StarRocksでColocate Join機能を使用するには、テーブルを作成する際にColocation Group（CG）を指定する必要があります。同じCG内のテーブルは、同じColocation Group Schema（CGS）に従う必要があります。つまり、テーブルに対応するバケットレプリカは、一貫したバケットキー、レプリカ数、レプリカの配置方法を持っている必要があります。これにより、同じCG内のすべてのテーブルのデータが同じBEノードグループに分布していることを保証できます。Join列がバケットキーである場合、計算ノードはローカルJoinのみを行う必要があり、ノード間のデータ転送時間を減らし、クエリのパフォーマンスを向上させることができます。したがって、Colocate Joinは、Shuffle JoinやBroadcast Joinなどの他のJoinと比較して、データネットワーク転送のコストを効果的に回避し、クエリのパフォーマンスを向上させることができます。

Colocate Joinは等価Joinをサポートしています。

## Colocate Join機能の使用

### Colocationテーブルの作成

テーブルを作成する際には、PROPERTIES内で`"colocate_with" = "group_name"`属性を指定してColocate Joinテーブルを作成し、特定のColocation Groupに属することを指定する必要があります。

> 注意
>
> バージョン2.5.4以降、異なるDatabase間でColocate Joinを実行することがサポートされました。テーブルを作成する際に同じ`colocate_with`属性を指定するだけです。

例：

~~~SQL
CREATE TABLE tbl (k1 int, v1 int sum)
DISTRIBUTED BY HASH(k1)
PROPERTIES(
    "colocate_with" = "group1"
);
~~~

指定されたCGが存在しない場合、StarRocksは自動的に現在のテーブルのみを含むCGを作成し、現在のテーブルをそのCGのParent Tableとして指定します。CGが既に存在する場合、StarRocksは現在のテーブルがCGSを満たしているかどうかをチェックします。満たしている場合、StarRocksはそのテーブルを作成し、Groupに追加します。同時に、StarRocksは既存のGroupのデータ分布ルールに基づいて現在のテーブルのためのシャードとレプリカを作成します。

Groupは一つのDatabaseに属しています。Group名はDatabase内でユニークであり、内部ストレージではGroupの完全名は`dbId_groupName`ですが、ユーザーは`groupName`のみを認識します。
> 注意
>
> 異なるDatabaseのテーブルに同じColocation Groupを指定して、これらのテーブルが相互にcolocateされるようにすると、そのColocation Groupは各Databaseに存在します。`show proc "/colocation_group"`コマンドを実行して、各Databaseに含まれるColocation Groupの情報を確認できます。

バケットキーのハッシュ値をバケット数で割ってバケットのシーケンス番号（Bucket Seq）を得ます。例えば、あるテーブルのバケット数が8の場合、\[0, 1, 2, 3, 4, 5, 6, 7\]の8つのバケットがあり、各バケットには1つまたは複数のサブテーブル（Tablet）があります。サブテーブルの数はテーブルのパーティション数に依存します：シングルパーティションテーブルの場合、各バケットには1つのサブテーブルのみがあります。マルチパーティションテーブルの場合、複数のサブテーブルがあります。

テーブルが同じデータ分布を持つためには、同じCG内のテーブルは以下の制約を満たす必要があります：

* 同じCG内のテーブルのバケットキーのタイプ、数、順序が完全に一致し、バケット数も一致している必要があります。これにより、複数のテーブルのデータシャードが一対一で対応する分布制御を行うことができます。バケットキーとは、`DISTRIBUTED BY HASH(col1, col2, ...)`で指定された列のセットです。バケットキーは、どの列の値を使用してテーブルのデータを異なるBucket Seqにハッシュ分割するかを決定します。同じCGのテーブルのバケットキーの名前は異なっても構いませんが、`DISTRIBUTED BY HASH(col1, col2, ...)`で指定された対応するデータタイプの順序は完全に一致している必要があります。
* 同じCG内のすべてのテーブルのすべてのパーティションのレプリカ数は一致している必要があります。一致していない場合、あるサブテーブルのレプリカが同じBEノード上に他のテーブルのシャードレプリカが対応していない可能性があります。
* 同じCG内のすべてのテーブルのパーティションキー、パーティション数は異なっても構いません。

同じCG内のすべてのテーブルのレプリカ配置は以下の制約を満たす必要があります：

* CG内のすべてのテーブルのBucket SeqとBEノードのマッピング関係はParent Tableと一致している必要があります。
* Parent Table内のすべてのパーティションのBucket SeqとBEノードのマッピング関係は最初のパーティションと一致している必要があります。
* Parent Tableの最初のパーティションのBucket SeqとBEノードのマッピング関係は、ネイティブのRound Robinアルゴリズムを使用して決定されます。

CG内のテーブルの一貫したデータ分布定義とサブテーブルレプリカのマッピングにより、バケットキーの値が同じデータ行は必ず同じBEノード上にあることが保証されるため、バケットキーをJoin列として使用する場合はローカルJoinのみが必要です。

### Colocation Groupの削除

Group内の最後のテーブルが完全に削除された後（完全に削除とは、ゴミ箱から削除されたことを意味します。通常、`DROP TABLE`コマンドで削除されたテーブルは、ゴミ箱でデフォルトで1日間保持された後、完全に削除されます）、そのGroupも自動的に削除されます。

### Group情報の確認

以下のコマンドを使用して、クラスタ内に既に存在するGroupの情報を確認できます。`root`ロールを持つユーザーのみが確認でき、一般ユーザーはサポートされていません。

~~~sql
SHOW PROC '/colocation_group';
~~~

例：

~~~Plain Text
mysql> SHOW PROC '/colocation_group';
+-------------+--------------+----------+------------+----------------+----------+----------+
| GroupId     | GroupName    | TableIds | BucketsNum | ReplicationNum | DistCols | IsStable |
+-------------+--------------+----------+------------+----------------+----------+----------+
| 11912.11916 | 11912_group1 | 11914    | 8          | 3              | int(11)  | true     |
+-------------+--------------+----------+------------+----------------+----------+----------+
~~~

|列名|説明|
|----|----|
|GroupId|Groupのクラスタ全体での一意の識別子。前半部分はDB ID、後半部分は**Group ID**です。|
|GroupName|Groupの完全名。|
|TableIds|そのGroupに含まれるテーブルのIDリスト。|
|BucketsNum|バケット数。|
|ReplicationNum|レプリカ数。|
|DistCols|Distribution columns、つまりバケット列のタイプ。|
|IsStable|そのGroupが[安定しているか](#colocation-副本均衡と修復)どうか。|

<br/>

以下のコマンドを使用して、特定のGroupのデータ分布状況をさらに確認できます。

~~~sql
SHOW PROC '/colocation_group/GroupId';
~~~

例：

~~~Plain Text
mysql> SHOW PROC '/colocation_group/11912.11916';
+-------------+---------------------+
| BucketIndex | BackendIds          |
+-------------+---------------------+
| 0           | 10002, 10004, 10003 |
| 1           | 10002, 10004, 10003 |
| 2           | 10002, 10004, 10003 |
| 3           | 10002, 10004, 10003 |
| 4           | 10002, 10004, 10003 |
| 5           | 10002, 10004, 10003 |
| 6           | 10002, 10004, 10003 |
| 7           | 10002, 10004, 10003 |
+-------------+---------------------+
8 rows in set (0.00 sec)
~~~

|項目名|説明|
|------|------|
|BucketIndex|バケットシーケンスのインデックス。|
|BackendIds|バケット内のデータシャードが配置されているBEノードのIDリスト。|

### テーブルGroup属性の変更

以下のコマンドを使用して、テーブルのColocation Group属性を変更できます。

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "group_name");
~~~

そのテーブルが以前にGroupを指定していなかった場合、このコマンドはスキーマをチェックし、そのテーブルをGroupに追加します（Groupが存在しない場合は作成されます）。そのテーブルが以前に別のGroupを指定していた場合、このコマンドはまずそのテーブルを元のGroupから削除し、新しいGroupに追加します（Groupが存在しない場合は作成されます）。

また、以下のコマンドを使用して、テーブルのColocation属性を削除することもできます：

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "");
~~~

<br/>

### その他の関連操作

Colocation属性を持つテーブルに対してパーティションを追加する`ADD PARTITION`操作やレプリカ数を変更する操作を行う場合、StarRocksはその操作がCGSに違反していないかをチェックします。違反している場合はその操作を拒否します。

## Colocationレプリカのバランスと修復

Colocationテーブルのレプリカ分布はGroupで指定された分布に従う必要があるため、通常のシャードと比較してレプリカの修復とバランスが異なります。

GroupにはIsStable属性があり、**IsStable**が**true**の場合（つまり安定状態）、現在のGroup内のテーブルのすべてのシャードに変更がないことを意味し、Colocation特性を正常に使用できます。**IsStable**が**false**の場合（つまり不安定状態）、現在のGroup内の一部のテーブルのシャードが修復または移行中であることを意味し、その場合、関連するテーブルのColocate Joinは通常のJoinに退化します。

### レプリカの修復

レプリカは指定されたBEノード上にのみ格納できるため、あるBEノードが利用できなくなった場合（例えばダウンまたはDecommissionなど）、StarRocksは新しいBEノードを見つけて置き換える必要があります。StarRocksは負荷が最も低いBEノードを優先して置き換えます。置き換え後、そのバケット内のすべての旧BEノード上のデータシャードは修復が必要です。移行中、Groupは不安定とマークされます。

### レプリカのバランス


StarRocks は、Colocation テーブルのシャードを可能な限り均等にすべての BE ノードに分散させます。通常のテーブルのレプリカバランスは、シングルレプリカを単位としており、つまり各レプリカに対して負荷の低い BE ノードを個別に見つけます。しかし、Colocation テーブルのバランスはバケットレベルであり、つまり一つのバケット内のすべてのレプリカが一緒に移動します。StarRocks は、レプリカの実際のサイズを考慮せず、レプリカの数に基づいて Bucket Seq をすべての BE ノードに均等に分散するという単純なバランスアルゴリズムを採用しています。具体的なアルゴリズムは `ColocateTableBalancer.java` のコードコメントを参照してください。

> 注意
>
> * 現在の Colocation レプリカのバランスと修復アルゴリズムは、異種デプロイされた StarRocks クラスターでは効果が理想的ではありません。異種デプロイとは、BE ノードのディスク容量、数量、ディスクタイプ（SSD と HDD）が一致しないことを指します。異種デプロイの状況下では、小容量の BE ノードと大容量の BE ノードが同じ数のレプリカを保持している可能性があります。
> * Group が Unstable 状態にある場合、その中のテーブルの Colocate Join は通常の Join に退化します。この時、クラスターのクエリ性能が大幅に低下する可能性があります。システムの自動バランスを望まない場合は、FE の設定項目 `tablet_sched_disable_colocate_balance` を設定して自動バランスを禁止することができます。その後、適切なタイミングで有効にすることができます。詳細は [高度な操作](#高度な操作) の節を参照してください。

## クエリ

Colocation テーブルのクエリ方法は通常のテーブルと同じで、ユーザーは Colocation 属性を意識する必要はありません。Colocation テーブルが属する Group が Unstable 状態の場合、自動的に通常の Join に退化します。

以下の例は、同じ CG 内のテーブル 1 とテーブル 2 を基にしています。

テーブル 1：

~~~SQL
CREATE TABLE `tbl1` (
    `k1` date NOT NULL COMMENT "",
    `k2` int(11) NOT NULL COMMENT "",
    `v1` int(11) SUM NOT NULL COMMENT ""
) ENGINE=OLAP
AGGREGATE KEY(`k1`, `k2`)
PARTITION BY RANGE(`k1`)
(
    PARTITION p1 VALUES LESS THAN ('2019-05-31'),
    PARTITION p2 VALUES LESS THAN ('2019-06-30')
)
DISTRIBUTED BY HASH(`k2`)
PROPERTIES (
    "colocate_with" = "group1"
);
~~~

テーブル 2：

~~~SQL
CREATE TABLE `tbl2` (
    `k1` datetime NOT NULL COMMENT "",
    `k2` int(11) NOT NULL COMMENT "",
    `v1` double SUM NOT NULL COMMENT ""
) ENGINE=OLAP
AGGREGATE KEY(`k1`, `k2`)
DISTRIBUTED BY HASH(`k2`)
PROPERTIES (
    "colocate_with" = "group1"
);
~~~

Join クエリプランを見る：

~~~Plain Text
EXPLAIN SELECT * FROM tbl1 INNER JOIN tbl2 ON (tbl1.k2 = tbl2.k2);

+----------------------------------------------------+
| Explain String                                     |
+----------------------------------------------------+
| PLAN FRAGMENT 0                                    |
|  OUTPUT EXPRS:`tbl1`.`k1` |                        |
|   PARTITION: RANDOM                                |
|                                                    |
|   RESULT SINK                                      |
|                                                    |
|   2:HASH JOIN                                      |
|   |  join op: INNER JOIN                           |
|   |  hash predicates:                              |
|   |  colocate: true                                |
|   |    `tbl1`.`k2` = `tbl2`.`k2`                   |
|   |  tuple ids: 0 1                                |
|   |                                                |
|   |----1:OlapScanNode                              |
|   |       TABLE: tbl2                              |
|   |       PREAGGREGATION: OFF. Reason: null        |
|   |       partitions=0/1                           |
|   |       rollup: null                             |
|   |       buckets=0/0                              |
|   |       cardinality=-1                           |
|   |       avgRowSize=0.0                           |
|   |       numNodes=0                               |
|   |       tuple ids: 1                             |
|   |                                                |
|   0:OlapScanNode                                   |
|      TABLE: tbl1                                   |
|      PREAGGREGATION: OFF. Reason: No AggregateInfo |
|      partitions=0/2                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 0                                  |
+----------------------------------------------------+
~~~

上記の例では、Hash Join ノードが `colocate: true` を表示し、Colocate Join が有効になっています。

以下の例では Colocate Join が有効になっていません：

~~~Plain Text
+----------------------------------------------------+
| Explain String                                     |
+----------------------------------------------------+
| PLAN FRAGMENT 0                                    |
|  OUTPUT EXPRS:`tbl1`.`k1` |                        |
|   PARTITION: RANDOM                                |
|                                                    |
|   RESULT SINK                                      |
|                                                    |
|   2:HASH JOIN                                      |
|   |  join op: INNER JOIN (BROADCAST)               |
|   |  hash predicates:                              |
|   |  colocate: false, reason: group is not stable  |
|   |    `tbl1`.`k2` = `tbl2`.`k2`                   |
|   |  tuple ids: 0 1                                |
|   |                                                |
|   |----3:EXCHANGE                                  |
|   |       tuple ids: 1                             |
|   |                                                |
|   0:OlapScanNode                                   |
|      TABLE: tbl1                                   |
|      PREAGGREGATION: OFF. Reason: No AggregateInfo |
|      partitions=0/2                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 0                                  |
|                                                    |
| PLAN FRAGMENT 1                                    |
|  OUTPUT EXPRS:                                     |
|   PARTITION: RANDOM                                |
|                                                    |
|   STREAM DATA SINK                                 |
|     EXCHANGE ID: 03                                |
|     UNPARTITIONED                                  |
|                                                    |
|   1:OlapScanNode                                   |
|      TABLE: tbl2                                   |
|      PREAGGREGATION: OFF. Reason: null             |
|      partitions=0/1                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 1                                  |
+----------------------------------------------------+
~~~

上記の例では、HASH JOIN ノードが Colocate Join が有効でないこととその理由を表示しています：`colocate: false, reason: group is not stable`。同時に StarRocks は EXCHANGE ノードを生成しています。

## 高度な操作

### FE 設定項目

`tablet_sched_disable_colocate_balance`：自動的な Colocation レプリカバランス機能を無効にするかどうか。デフォルトは `false` で、無効にしません。このパラメータは Colocation テーブルのレプリカバランスにのみ影響し、通常のテーブルには影響しません。

上記のパラメータは動的に変更が可能で、以下のコマンドで無効にすることができます。

~~~sql
ADMIN SET FRONTEND CONFIG ("tablet_sched_disable_colocate_balance" = "TRUE");
~~~

### Session 変数

`disable_colocate_join`：セッションの粒度で Colocate Join 機能を無効にするかどうか。デフォルトは `false` で、無効にしません。

上記のパラメータは動的に変更が可能で、以下のコマンドで無効にすることができます。

~~~sql
SET disable_colocate_join = TRUE;
~~~

### HTTP Restful API

StarRocks は、Colocate Join に関連する複数の HTTP Restful API を提供しており、Colocation Group の表示と変更に使用されます。

この API は FE で実装されており、`fe_host:fe_http_port` を使用してアクセスできます。アクセスには `cluster_admin` ロールの権限が必要です。

1. クラスターのすべての Colocation 情報を表示します。

    ~~~bash
    curl --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate'  
    ~~~

    例：

    ~~~JSON
    // 内部の Colocation 情報を Json 形式で返します。
    {
        "colocate_meta": {
            "groupName2Id": {
                "g1": {
                    "dbId": 10005,
                    "grpId": 10008
                }
            },
            "group2Tables": {},
            "table2Group": {
                "10007": {
                    "dbId": 10005,
                    "grpId": 10008
                },
                "10040": {
                    "dbId": 10005,
                    "grpId": 10008
                }
            },
            "group2Schema": {
                "10005.10008": {
                    "groupId": {
                        "dbId": 10005,
                        "grpId": 10008
                    },
                    "distributionColTypes": [{
                        "type": "INT",
                        "len": -1,
                        "isAssignedStrLenInColDefinition": false,
                        "precision": 0,
                        "scale": 0
                    }],
                    "bucketsNum": 10,
                    "replicationNum": 2
                }
            },
            "group2BackendsPerBucketSeq": {
                "10005.10008": [
                    [10004, 10002],
                    [10003, 10002],
                    [10002, 10004],
                    [10003, 10002],
                    [10002, 10004],
                    [10003, 10002],
                    [10003, 10004],
                    [10003, 10004],
                    [10003, 10004],
                    [10002, 10004]
                ]
            },
            "unstableGroups": []
        },
        "status": "OK"
    }
    ~~~

2. Group を Stable または Unstable としてマークします。

    ~~~shell
    # Stable としてマーク。
    curl -XPOST --location-trusted -u<user>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_stable?db_id=<dbId>&group_id=<grpId>'

    # Unstable としてマーク。
    curl -XPOST --location-trusted -u<user>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_unstable?db_id=<dbId>&group_id=<grpId>'
    ~~~

    返答が `200` であれば、マークの変更が成功したことを意味します。

3. Group のデータ分布を設定します。

    このインターフェースを使用して、特定の Group のデータ分布を強制的に設定できます。

    > 注意
    > このコマンドを使用するには、FE の設定 `tablet_sched_disable_colocate_balance` を `true` に設定して、システムの自動 Colocation レプリカ修復とバランスを無効にする必要があります。そうしないと、データ分布の設定を変更した後、システムによって自動的にリセットされる可能性があります。

    ~~~shell
    curl -u<user>:<password> -X POST "http://<fe_host>:<fe_http_port>/api/colocate/bucketseq?db_id=10005&group_id=10008"
    ~~~

    `Body:`


    `[[10004,10002],[10003,10002],[10002,10004],[10003,10002],[10002,10004],[10003,10002],[10003,10004],[10003,10004],[10003,10004],[10002,10004]]`

    `レスポンス：200`

    ここで Body は、ネストされた配列として表される Bucket Seq および各バケット内のシャードが存在する BE の ID を示しています。


---
displayed_sidebar: Chinese
---
# EXPORTデータの使用

この記事では、[EXPORT](../sql-reference/sql-statements/data-manipulation/EXPORT.md)ステートメントを使用して、指定されたテーブルまたはパーティションのStarRocksクラスタのデータをCSV形式で外部ストレージシステムにエクスポートする方法について説明します。現在、HDFSまたはAWS S3、Alibaba Cloud OSS、Tencent Cloud COS、Huawei Cloud OBSなどのクラウドストレージシステムにエクスポートすることができます。

> **注意**
>
> エクスポート操作には、ターゲットテーブルのEXPORT権限が必要です。ユーザーアカウントにEXPORT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。

## 背景情報

v2.4およびそれ以前のバージョンでは、StarRocksはデータのエクスポートにBrokerを使用して外部ストレージシステムにアクセスする必要がありました。これは「Broker付きエクスポート」と呼ばれます。エクスポートステートメントでは、`WITH BROKER "<broker_name>"`を使用してどのBrokerを使用するかを指定する必要がありました。Brokerは、ファイルシステムインターフェースをカプセル化した独立したステートレスサービスであり、StarRocksがデータを外部ストレージシステムにエクスポートするのを支援します。

v2.5以降、StarRocksはBrokerなしで外部ストレージシステムにアクセスするためのEXPORTステートメントを使用できるようになりました。エクスポートステートメントでは、`broker_name`を指定する必要はありませんが、`WITH BROKER`キーワードは引き続き使用されます。

ただし、HDFSをデータソースとする場合、Brokerなしのエクスポートは制限される場合があります。この場合は、引き続きBroker付きのエクスポートを実行できます。以下の場合に該当します。

- 複数のHDFSクラスタが構成されている場合、各HDFSクラスタに独立のBrokerをデプロイする必要があります。
- 単一のHDFSクラスタが構成されているが、複数のKerberosユーザーが存在する場合、独立したBrokerを1つデプロイするだけで十分です。

> **注意**
>
> クラスタにデプロイされているBrokerを表示するには、[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)ステートメントを使用できます。クラスタにBrokerがデプロイされていない場合は、[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してBrokerのデプロイを完了してください。

## サポートされている外部ストレージシステム

- 分散ファイルシステムHDFS
- AWS S3、Alibaba Cloud OSS、Tencent Cloud COS、Huawei Cloud OBSなどのクラウドストレージシステム

## 注意事項

- 大量のデータを一度にエクスポートしないことをお勧めします。エクスポートジョブの推奨データ量は数十GBです。一度に大量のデータをエクスポートすると、エクスポートが失敗する可能性があり、再試行のコストも増加します。

- テーブルのデータ量が大きい場合は、パーティションごとにエクスポートすることをお勧めします。

- エクスポートジョブの実行中にFEが再起動またはメインを切り替えると、エクスポートジョブが失敗し、エクスポートジョブを再提出する必要があります。

- エクスポートジョブの実行が完了した後（成功または失敗）、FEが再起動またはメインを切り替えると、[SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)ステートメントで返されるエクスポートジョブの情報が一部失われ、表示できなくなります。

- エクスポートジョブは元のテーブル（ベーステーブル）のデータのみをエクスポートし、マテリアライズドビューのデータはエクスポートしません。

- エクスポートジョブはデータをスキャンし、I/Oリソースを使用するため、システムのクエリ遅延に影響を与える可能性があります。

## エクスポートのプロセス

エクスポートジョブを提出すると、StarRocksはそのジョブに関連するすべてのタブレットを集計し、これらのタブレットをグループ化し、各グループに特別なクエリプランを生成します。このクエリプランは、含まれるタブレット上のデータを読み取り、指定されたパスにデータを書き込む役割を担当します。

エクスポートジョブの全体的な処理フローは、次の図に示すようになります。

![エクスポートジョブのフローチャート](../assets/5.1-2.png)

エクスポートジョブの全体的な処理フローは、次の3つのステップで構成されています。

1. ユーザーはリーダーFEにエクスポートジョブを提出します。
2. リーダーFEはクラスタ内のすべてのBEに`snapshot`コマンドを送信し、関連するすべてのタブレットにスナップショットを作成し、データの一貫性を保ち、複数のエクスポートサブタスクを生成します。各サブタスクはクエリプランであり、各クエリプランは一部のタブレットを処理します。
3. リーダーFEは1つずつエクスポートサブタスクをBEに送信して実行します。

## 基本原理

クエリプランを実行する際、StarRocksはまず指定されたリモートストレージのパスに、`__starrocks_export_tmp_xxx`という名前の一時ディレクトリを作成します。ここで、`xxx`はエクスポートジョブのクエリIDです。たとえば、`__starrocks_export_tmp_921d8f80-7c9d-11eb-9342-acde48001122`となります。各クエリプランが正常に実行されると、エクスポートされたデータはまずこの一時ディレクトリに書き込まれます。

すべてのデータがエクスポートされた後、StarRocksはRENAMEステートメントを使用してこれらのファイルを指定されたパスに保存します。

## 関連する設定

ここでは、データのエクスポートに関連するいくつかのFE設定パラメータについて説明します。

- `export_checker_interval_second`：エクスポートジョブスケジューラのスケジュール間隔。デフォルトは5秒です。このパラメータを設定するには、FEを再起動する必要があります。
- `export_running_job_num_limit`：実行中のエクスポートジョブの数の制限。この制限を超えると、ジョブは`snapshot`の実行後に待機状態になります。デフォルトは5です。エクスポートジョブの実行時にこのパラメータの値を調整することができます。
- `export_task_default_timeout_second`：エクスポートジョブのタイムアウト時間。デフォルトは2時間です。エクスポートジョブの実行時にこのパラメータの値を調整することができます。
- `export_max_bytes_per_be_per_task`：各エクスポートサブタスクが各BEでエクスポートする最大データ量。圧縮後のデータ量で計算されます。デフォルトは256MBです。
- `export_task_pool_size`：エクスポートサブタスクのスレッドプールのサイズ、つまり並行して実行できる最大サブタスク数です。デフォルトは5です。

## 基本的な操作

### エクスポートジョブの提出

次のコマンドを使用して、データベース`db1`のテーブル`tbl1`の`p1`および`p2`パーティション上の`col1`および`col3`のデータをHDFSストレージの`export/lineorder_`ディレクトリにエクスポートできます。

```SQL
EXPORT TABLE db1.tbl1 
PARTITION (p1,p2)
(col1, col3)
TO "hdfs://HDFS_IP:HDFS_Port/export/lineorder_" 
PROPERTIES
(
    "column_separator"=",",
    "load_mem_limit"="2147483648",
    "timeout" = "3600"
)
WITH BROKER
(
    "username" = "user",
    "password" = "passwd"
);
```

EXPORTステートメントの詳細な構文とパラメータの説明、およびAWS S3、Alibaba Cloud OSS、Tencent Cloud COS、Huawei Cloud OBSなどのクラウドストレージシステムにデータをエクスポートするコマンドの例については、[EXPORT](../sql-reference/sql-statements/data-manipulation/EXPORT.md)を参照してください。

### エクスポートジョブのクエリIDの取得

エクスポートジョブを提出した後、SELECT LAST_QUERY_ID()ステートメントを使用してエクスポートジョブのクエリIDを取得できます。このIDを使用してエクスポートジョブを表示またはキャンセルすることができます。

SELECT LAST_QUERY_ID()ステートメントの詳細な構文とパラメータの説明については、[last_query_id](../sql-reference/sql-functions/utility-functions/last_query_id.md)を参照してください。

### エクスポートジョブのステータスの表示

エクスポートジョブを提出した後、SHOW EXPORTステートメントを使用してエクスポートジョブのステータスを表示できます。次のように指定します。

```SQL
SHOW EXPORT WHERE queryid = "edee47f0-abe1-11ec-b9d1-00163e1e238f";
```

> **注意**
>
> 上記の例では、`queryid`はエクスポートジョブのIDです。

システムは次のようなエクスポート結果を返します。

```Plain_Text
JobId: 14008
State: FINISHED
Progress: 100%
TaskInfo: {"partitions":["*"],"mem limit":2147483648,"column separator":",","line delimiter":"\n","tablet num":1,"broker":"hdfs","coord num":1,"db":"default_cluster:db1","tbl":"tbl3",columns:["col1", "col3"]}
Path: oss://bj-test/export/
CreateTime: 2019-06-25 17:08:24
StartTime: 2019-06-25 17:08:28
FinishTime: 2019-06-25 17:08:34
Timeout: 3600
ErrorMsg: N/A
```

SHOW EXPORTステートメントの詳細な構文とパラメータの説明については、[SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)を参照してください。

### エクスポートジョブのキャンセル

エクスポートジョブを提出した後、エクスポートジョブが完了する前にCANCEL EXPORTステートメントを使用してエクスポートジョブをキャンセルすることができます。次のように指定します。

```SQL
CANCEL EXPORT WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```

> **注意**
>
> 上記の例では、`queryid`はエクスポートジョブのIDです。

CANCEL EXPORTステートメントの詳細な構文とパラメータの説明については、[CANCEL EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)を参照してください。

## ベストプラクティス

### クエリプランの分割

エクスポートジョブには、実行するクエリプランの数がいくつあるか、およびクエリプランが処理できる最大データ量がどれくらいかによって決まります。エクスポートジョブはクエリプランごとにリトライされます。クエリプランが処理するデータ量が許容される最大データ量を超えると、クエリプランが失敗します（たとえば、BrokerのRPCが失敗した場合、リモートストレージに障害が発生した場合など）。これにより、クエリプランのリトライコストが高くなります。各クエリプランで各BEがスキャンするデータ量は、FEの設定パラメータ`export_max_bytes_per_be_per_task`で設定されます。デフォルト値は256MBです。各クエリプランで各BEには少なくとも1つのタブレットが割り当てられ、エクスポートされるデータ量は`export_max_bytes_per_be_per_task`の値を超えません。

エクスポートジョブの複数のクエリプランは並行して実行され、サブタスクスレッドプールのサイズはFEの設定パラメータ`export_task_pool_size`で設定されます。デフォルト値は5です。

通常、エクスポートジョブのクエリプランには「スキャン」と「エクスポート」の2つのパートがあり、計算ロジックはあまりメモリを消費しません。したがって、デフォルトの2GBのメモリ制限は要件を満たすことができます。ただし、特定のシナリオでは、1つのクエリプランでスキャンするタブレットが多すぎる場合や、タブレットのデータバージョンが多すぎる場合など、メモリが不足する可能性があります。この場合は、`load_mem_limit`パラメータを変更して、より大きなメモリ（たとえば4GB、8GBなど）を設定する必要があります。

```
---
displayed_sidebar: Chinese
---

# 監視アラーム

この文書では、StarRocksに監視アラームを設定する方法について説明します。

StarRocksは2種類の監視アラーム方案を提供しています。エンタープライズ版ユーザーは、内蔵のStarRocksManagerを使用できます。そのAgentは各Hostから監視情報を収集し、Center Serviceに報告してから可視化表示を行います。StarRocksManagerはメールとWebhookによるアラーム通知を提供します。二次開発のニーズがある場合は、監視サービスを自分で構築してデプロイすることもできますし、オープンソースのPrometheus+Grafana方案を使用することもできます。StarRocksはPrometheusと互換性のある情報収集インターフェースを提供しており、BEまたはFEのHTTPポートに直接接続してクラスタの監視情報を取得することができます。

## StarRocksManagerの使用

StarRocksManagerの監視は、**クラスタ**と**ノード**の2つの次元に分けることができます。

クラスタページでは、以下の監視項目を確認できます：

* クラスタ性能監視
  * CPU使用率
  * メモリ使用
  * ディスクI/O使用率、ディスク使用量、ディスク空き容量
  * パケット送信帯域幅、パケット受信帯域幅、パケット送信数、パケット受信数
* クラスタクエリ監視
  * QPS
  * 平均応答時間
  * 50/75/90/95/99/999パーセンタイル応答時間
* データインポート量監視
  * インポート開始回数
  * インポート行数
  * インポートデータ量
* データコンパクション（Compaction）監視
  * ベースラインコンパクションデータグループ速度
  * ベースラインコンパクションデータ量
  * インクリメンタルコンパクションデータグループ速度
  * インクリメンタルコンパクションデータ量

**ノード**ページでは、すべてのBE/FEのマシンリストと状態などの基本情報を確認できます。

![8.10.1-1](../assets/8.10.1-1.png)

ノードリンクをクリックすると、各ノードの詳細な監視情報が表示されます。右側のノードリストから複数のノードを選択して同時に表示したり、上部のドロップダウンリストから各種指標を選択することもできます。

![8.10.1-2](../assets/8.10.1-2.png)

## Prometheus+Grafanaの使用

[Prometheus](https://prometheus.io/)をStarRocksの監視データストレージ方案として使用し、[Grafana](https://grafana.com/)を可視化コンポーネントとして使用することができます。

Prometheusは、多次元データモデルと柔軟なクエリ言語を持つ時系列データベースです。PullまたはPushで監視対象システムの監視項目を収集し、自身の時系列データベースに格納します。豊富な多次元データクエリ言語を通じて、ユーザーのさまざまなニーズを満たします。

Grafanaは、オープンソースのメトリック分析および可視化システムです。多種多様なデータソースをサポートしており、詳細は公式ドキュメントを参照してください。対応するクエリ言語を通じてデータソースから表示データを取得します。柔軟に設定可能なダッシュボードを通じて、これらのデータを迅速にグラフ形式でユーザーに表示します。

> この文書は、PrometheusとGrafanaを使用したStarRocksの可視化監視方案の一例を提供するのみであり、原則としてこれらのコンポーネントのメンテナンスや開発は行いません。より詳細な紹介と使用については、対応する公式ドキュメントを参照してください。

### 監視アーキテクチャ

![8.10.2-1](../assets/8.10.2-1.png)

PrometheusはPull方式でFEまたはBEのMetricインターフェースにアクセスし、監視データを時系列データベースに格納します。

ユーザーはGrafanaでPrometheusをデータソースとして設定し、カスタムダッシュボードを描画することができます。

### Prometheusのデプロイ

#### Prometheusのダウンロードとインストール

**1.** [Prometheusの公式ウェブサイト](https://prometheus.io/download/)から最新バージョンのPrometheusをダウンロードします。

以下の例では、prometheus-2.29.1.linux-amd64バージョンを使用しています。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

#### Prometheusの設定

**prometheus.yml**にStarRocksの監視に関連する設定を追加します。

```yml
# my global config
global:
  scrape_interval:     15s # Set the global scraping interval to 15s, default is 1m
  evaluation_interval: 15s # Set the global rule evaluation interval to 15s, default is 1m

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'StarRocks_Cluster01' # Each cluster is called a job, you can customize the name as the StarRocks cluster name
    metrics_path: '/metrics'    # Specify the Restful Api to get monitoring items

    static_configs:
      - targets: ['fe_host1:http_port','fe_host2:http_port','fe_host3:http_port']
        labels:
          group: fe # This configuration includes 3 Frontends in the fe group

      - targets: ['be_host1:be_http_port', 'be_host2:be_http_port', 'be_host3:be_http_port']
        labels:
          group: be # This configuration includes 3 Backends in the be group
  - job_name: 'StarRocks_Cluster02' # Multiple StarRocks clusters can be monitored in Prometheus
    metrics_path: '/metrics'

    static_configs:
      - targets: ['fe_host1:http_port','fe_host2:http_port','fe_host3:http_port']
        labels:
          group: fe

      - targets: ['be_host1:be_http_port', 'be_host2:be_http_port', 'be_host3:be_http_port']
        labels:
          group: be
```

#### Prometheusの起動

以下のコマンドでPrometheusを起動します。

```bash
nohup ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --log.level="info" &
```

このコマンドはPrometheusをバックグラウンドで実行し、Webポートを`9090`に指定します。起動後、データの収集を開始し、データは**./data**ディレクトリに格納されます。

#### Prometheusへのアクセス

WebページからPrometheusにアクセスできます。ブラウザで`9090`ポートを開くと、Prometheusのページにアクセスできます。ナビゲーションバーの**Status**と**Targets**を順にクリックすると、すべてのグループ化されたJobの監視ホストノードが表示されます。通常、すべてのノードはUPであるべきです。これはデータ収集が正常であることを意味します。ノードの状態がUPでない場合は、StarRocksのMetricsインターフェース(`http://fe_host:fe_http_port/metrics`または`http://be_host:be_http_port/metrics`)にアクセスしてアクセス可能かどうかを確認できます。それでも解決できない場合は、Prometheusの関連ドキュメントを参照して解決策を探すことができます。

![8.10.2-6](../assets/8.10.2-6.png)

これで、シンプルなPrometheusのセットアップと設定が完了しました。より高度な使用方法については、[公式ドキュメント](https://prometheus.io/docs/introduction/overview/)を参照してください。

#### 既存のPrometheusにデータを接続する

監視データを既存のPrometheusシステムに接続する必要がある場合は、以下のインターフェースを通じてアクセスできます：

* FE: `fe_host:fe_http_port/metrics`
* BE: `be_host:be_http_port/metrics`

JSON形式のデータが必要な場合は、以下のインターフェースを通じてアクセスできます：

* FE: `fe_host:fe_http_port/metrics?type=json`
* BE: `be_host:be_http_port/metrics?type=json`

### Grafanaのデプロイ

#### Grafanaのダウンロードとインストール

[Grafanaの公式ウェブサイト](https://grafana.com/grafana/download)から最新バージョンのGrafanaをダウンロードします。

以下の例では、grafana-8.0.6.linux-amd64バージョンを使用しています。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

#### Grafanaの設定

**./conf/defaults.ini**に関連する設定を追加します。

```ini
...
[paths]
data = ./data
logs = ./data/log
plugins = ./data/plugins
[server]
http_port = 8000
domain = localhost
...
```

#### Grafanaの起動

以下のコマンドでGrafanaを起動します。

```Plain text
nohup ./bin/grafana-server \
    --config="./conf/defaults.ini" &
```

#### ダッシュボードの設定

以前に設定したアドレス(`http://grafana_host:8000`)からGrafanaにログインします。デフォルトのユーザー名は`admin`、パスワードは`admin`です。

**1.** データソースを設定します。

**Configuration**、**Data sources**、**Add data source**、**Prometheus**の順にクリックします。

データソース設定項目の概要

![8.10.2-2](../assets/8.10.2-2.png)

* 名前: データソースの名前は自由に設定できます。例えば`starrocks_monitor`などです。

* URL: Prometheusのwebアドレス、例えば `http://prometheus_host:9090`
* Access: Serverを選択し、Grafanaプロセスが稼働しているサーバー経由でPrometheusにアクセスします。

他の設定項目はデフォルトのまま使用できます。

最下部の **Save & Test** をクリックして設定を保存し、*Data source is working* と表示されれば、データソースが利用可能であることを意味します。

**2.** ダッシュボードの追加。

ダッシュボードテンプレートをダウンロードします。

> 説明：StarRocks 1.19.0 および 2.4.0 のバージョンでは監視 Metric Name が調整されているため、以下の対応するバージョンのダッシュボードテンプレートをダウンロードする必要があります。

* [StarRocks-1.19.0 以前のバージョンのダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
* [StarRocks-1.19.0 から StarRocks-2.4.0 以前のバージョンのダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
* [StarRocks-2.4.0 以降のバージョンのダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24-new.json)

> 説明：ダッシュボードテンプレートは定期的に更新されます。また、優れたダッシュボードテンプレートの提供も歓迎します。

データソースが利用可能であることを確認した後、左側のナビゲーションバーの **+** をクリックしてダッシュボードを追加します。ここでは、先にダウンロードしたStarRocksのダッシュボードテンプレートを使用します。左側のナビゲーションバーの **+** 、**Import**、**Upload Json File** の順にクリックし、JSONファイルをインポートします。

インポート後、ダッシュボードに名前を付けることができ、デフォルトでは `StarRocks Overview` です。同時に、データソースを選択する必要があり、ここでは以前に作成した `starrocks_monitor` を選択します。

**Import** をクリックしてインポートを完了します。これで、StarRocksのダッシュボードが表示されます。

#### ダッシュボードを理解する

このセクションでは、StarRocksダッシュボードについて簡単に説明します。

> 注意：ダッシュボードの内容はバージョンアップに伴い更新される可能性があるため、上記のダッシュボードテンプレートを参照してください。

* **トップバー**

![8.10.2-3](../assets/8.10.2-3.png)

ページの左上隅にはダッシュボードの名前があり、右上隅には現在の監視時間範囲が表示されます。異なる時間範囲を選択するドロップダウンがあり、定期的なページのリフレッシュ間隔を設定することもできます。

* cluster_name: Prometheusの設定ファイル内の `job_name` に対応し、StarRocksクラスターを表します。異なるClusterを選択すると、下のグラフには選択したクラスターの監視情報が表示されます。
* fe_master: クラスターのLeader FEノード。
* fe_instance: クラスターのすべてのFEノード。異なるFEを選択すると、下のグラフには選択したFEの監視情報が表示されます。
* be_instance: クラスターのすべてのBEノード。異なるBEを選択すると、下のグラフには選択したBEの監視情報が表示されます。
* interval: 一部のグラフでは、特定の間隔でサンプリングして速度を計算する速度関連の監視項目が表示されます。

> 注意：15秒を時間間隔として使用しないことをお勧めします。これにより、一部のグラフが表示されない可能性があります。

* **Row**

![8.10.2-4](../assets/8.10.2-4.png)

Grafanaでは、Rowは一連のグラフの集合を表します。上の画像のOverview、Cluster Overviewは異なる2つのRowです。現在のRowをクリックしてそのRowを折りたたむことができます。

現在のダッシュボードには以下のRowがあります（更新中）：

* Overview: すべてのStarRocksクラスターの集約表示。
* Cluster Overview: 選択したクラスターの集約表示。
* Query Statistic: 選択したクラスターのクエリ関連の監視。
* Jobs: インポートジョブ関連の監視。
* Transaction: トランザクション関連の監視。
* FE JVM: 選択したFEのJVM監視。
* BE: 選択したクラスターのBEの集約表示。
* BE Task: 選択したクラスターのBEのタスク情報表示。

* **グラフ**

![8.10.2-5](../assets/8.10.2-5.png)

典型的なグラフは以下の部分で構成されています：

* 左上角の **i** アイコンにマウスを合わせると、そのグラフの説明を見ることができます。
* 下の凡例をクリックすると、特定の監視項目だけを表示することができます。もう一度クリックすると、すべてが表示されます。
* グラフ内でドラッグすると、時間範囲を選択することができます。
* タイトルの **[]** 内には選択したクラスターの名前が表示されます。
* 一部の数値は左側のY軸に対応し、一部は右側のY軸に対応しています。凡例の末尾の **-right** で区別することができます。
* **グラフ名** と **Edit** を順にクリックすると、グラフを編集することができます。

## 監視指標

選択可能なStarRocks監視指標は以下の通りです：

|指標|単位|タイプ|説明|
|---|:---:|:---:|---|
|be_broker_count|個|平均値|Brokerの数。|
|be_brpc_endpoint_count|個|平均値|bRPCのStubCacheの数。|
|be_bytes_read_per_second|bytes/s|平均値|BEの読み取り速度。|
|be_bytes_written_per_second|bytes/s|平均値|BEの書き込み速度。|
|be_base_compaction_bytes_per_second|bytes/s|平均値|BEのベースコンパクション速度。|
|be_cumulative_compaction_bytes_per_second|bytes/s|平均値|BEの累積コンパクション速度。|
|be_base_compaction_rowsets_per_second|rowsets/s|平均値|BEのベースコンパクションのrowsets合成速度。|
|be_cumulative_compaction_rowsets_per_second|rowsets/s|平均値|BEの累積コンパクションのrowsets合成速度。|
|be_base_compaction_failed|個/秒|平均値|BEのベースコンパクション失敗。|
|be_clone_failed|個/秒|平均値|BEのクローン失敗。|
|be_create_rollup_failed|個/秒|平均値|BEのロールアップ作成失敗。|
|be_create_tablet_failed|個/秒|平均値|BEのタブレット作成失敗。|
|be_cumulative_compaction_failed|個/秒|平均値|BEの累積コンパクション失敗。|
|be_delete_failed|個/秒|平均値|BEの削除失敗。|
|be_finish_task_failed|個/秒|平均値|BEのタスク失敗。|
|be_publish_failed|個/秒|平均値|BEのバージョン公開失敗。|
|be_report_tables_failed|個/秒|平均値|BEのテーブル報告失敗。|
|be_report_disk_failed|個/秒|平均値|BEのディスク報告失敗。|
|be_report_tablet_failed|個/秒|平均値|BEのタブレット報告失敗。|
|be_report_task_failed|個/秒|平均値|BEのタスク報告失敗。|
|be_schema_change_failed|個/秒|平均値|BEのスキーマ変更失敗。|
|be_base_compaction_requests|個/秒|平均値|BEのベースコンパクションリクエスト。|
|be_clone_total_requests|個/秒|平均値|BEのクローンリクエスト。|
|be_create_rollup_requests|個/秒|平均値|BEのロールアップ作成リクエスト。|
|be_create_tablet_requests|個/秒|平均値|BEのタブレット作成リクエスト。|
|be_cumulative_compaction_requests|個/秒|平均値|BEの累積コンパクションリクエスト。|
|be_delete_requests|個/秒|平均値|BEの削除リクエスト。|
|be_finish_task_requests|個/秒|平均値|BEのタスク完了リクエスト。|
|be_publish_requests|個/秒|平均値|BEのバージョン公開リクエスト。|
|be_report_tablets_requests|個/秒|平均値|BEのタブレット報告リクエスト。|
|be_report_disk_requests|個/秒|平均値|BEのディスク報告リクエスト。|
|be_report_tablet_requests|個/秒|平均値|BEのタブレット報告リクエスト。|
|be_report_task_requests|個/秒|平均値|BEのタスク報告リクエスト。|
|be_schema_change_requests|個/秒|平均値|BEのスキーマ変更リクエスト。|
|be_storage_migrate_requests|個/秒|平均値|BEのストレージ移行リクエスト。|
|be_fragment_endpoint_count|個|平均値|BEのDataStreamの数。|
|be_fragment_request_latency_avg|ms|平均値|fragmentリクエストの応答時間。|
|be_fragment_requests_per_second|個/秒|平均値|fragmentリクエスト数。|
|be_http_request_latency_avg|ms|平均値|HTTPリクエストの応答時間。|
|be_http_requests_per_second|個/秒|平均値|HTTPリクエスト数。|
|be_http_request_send_bytes_per_second|bytes/s|平均値|HTTPリクエストの送信バイト数。|
|fe_connections_per_second|接続数/s|平均値|FEの新規接続速度。|
|fe_connection_total|接続数|累計値|FEの総接続数。|
|fe_edit_log_read|操作数/s|平均値|FEのedit log読み取り速度。|
|fe_edit_log_size_bytes|bytes/s|平均値|FE edit log のサイズ。|
|fe_edit_log_write|bytes/s|平均値|FE edit log の書き込み速度。|
|fe_checkpoint_push_per_second|operations/s|平均値|FEチェックポイント数。|
|fe_pending_hadoop_load_job|個|平均値|Pending 状態の Hadoop job 数。|
|fe_committed_hadoop_load_job|個|平均値|コミット済みの Hadoop job 数。|
|fe_loading_hadoop_load_job|個|平均値|ローディング中の Hadoop job 数。|
|fe_finished_hadoop_load_job|個|平均値|完了した Hadoop job 数。|
|fe_cancelled_hadoop_load_job|個|平均値|キャンセルされた Hadoop job 数。|
|fe_pending_insert_load_job|個|平均値|Pending 状態の insert job 数。|
|fe_loading_insert_load_job|個|平均値|ローディング中の insert job 数。|
|fe_committed_insert_load_job|個|平均値|コミット済みの insert job 数。|
|fe_finished_insert_load_job|個|平均値|完了した insert job 数。|
|fe_cancelled_insert_load_job|個|平均値|キャンセルされた insert job 数。|
|fe_pending_broker_load_job|個|平均値|Pending 状態の broker job 数。|
|fe_loading_broker_load_job|個|平均値|ローディング中の broker job 数。|
|fe_committed_broker_load_job|個|平均値|コミット済みの broker job 数。|
|fe_finished_broker_load_job|個|平均値|完了した broker job 数。|
|fe_cancelled_broker_load_job|個|平均値|キャンセルされた broker job 数。|
|fe_pending_delete_load_job|個|平均値|Pending 状態の delete job 数。|
|fe_loading_delete_load_job|個|平均値|ローディング中の delete job 数。|
|fe_committed_delete_load_job|個|平均値|コミット済みの delete job 数。|
|fe_finished_delete_load_job|個|平均値|完了した delete job 数。|
|fe_cancelled_delete_load_job|個|平均値|キャンセルされた delete job 数。|
|fe_rollup_running_alter_job|個|平均値|実行中の Rollup 変更 job 数。|
|fe_schema_change_running_job|個|平均値|スキーマ変更中の job 数。|
|cpu_util| パーセント|平均値|CPU 使用率。|
|cpu_system | パーセント|平均値|cpu_system 使用率。|
|cpu_user| パーセント|平均値|cpu_user 使用率。|
|cpu_idle| パーセント|平均値|cpu_idle 使用率。|
|cpu_guest| パーセント|平均値|cpu_guest 使用率。|
|cpu_iowait| パーセント|平均値|cpu_iowait 使用率。|
|cpu_irq| パーセント|平均値|cpu_irq 使用率。|
|cpu_nice| パーセント|平均値|cpu_nice 使用率。|
|cpu_softirq| パーセント|平均値|cpu_softirq 使用率。|
|cpu_steal| パーセント|平均値|cpu_steal 使用率。|
|disk_free|bytes|平均値|空きディスク容量。|
|disk_io_svctm|ms|平均値|ディスク IO サービス時間。|
|disk_io_util|パーセント|平均値|ディスク使用率。|
|disk_used|bytes|平均値|使用済みディスク容量。|
|starrocks_fe_query_resource_group|個|累積値|該当リソースグループ内のクエリタスク数。|
|starrocks_fe_query_resource_group_latency|秒|平均値|該当リソースグループのクエリ遅延パーセンタイル。|
|starrocks_fe_query_resource_group_err|個|累積値|該当リソースグループ内でエラーが発生したクエリタスク数。|
|starrocks_fe_meta_log_count|個|瞬時値|Checkpoint 未実施の Edit Log 数。この値は `100000` 以内であれば適切。|
|starrocks_be_resource_group_cpu_limit_ratio|パーセント|瞬時値|該当リソースグループの CPU クォータ比率の瞬時値。|
|starrocks_be_resource_group_cpu_use_ratio|パーセント|平均値|該当リソースグループの CPU 使用時間が全リソースグループの CPU 使用時間に占める割合。|
|starrocks_be_resource_group_mem_limit_bytes|Bytes|瞬時値|該当リソースグループのメモリクォータ比率の瞬時値。|
|starrocks_be_resource_group_mem_allocated_bytes|Bytes|瞬時値|該当リソースグループのメモリ使用率の瞬時値。|
|starrocks_be_pipe_prepare_pool_queue_len|個|瞬時値|Pipeline 準備スレッドプールのタスクキュー長の瞬時値。|

## ベストプラクティス

監視システムは15秒ごとに情報を収集します。一部の指標は15秒間隔の監視情報の平均値であり、単位は「個/秒」です。他の指標は15秒の総値に基づいています。

現在の P90、P99 などのパーセンタイル監視情報は、15秒間隔に基づいています。より大きな粒度（1分、5分など）で監視アラートを設定する場合、システムは「何回発生したか」または「どれだけの値を超えたか」を指標として使用し、「平均値はいくらか」ではありません。

現在のベストプラクティスは以下の共通認識に基づいています：

* 監視は異常状態でアラートを出すべきであり、正常状態でのアラートを避けるべきです。
* 異なるクラスターのリソース（例えばメモリ、ディスク）の使用量が異なるため、異なる値を設定する必要があります。このようなゲージ値はパーセンテージを単位として使用する方が普遍的です。
* タスク失敗回数などの監視情報については、タスクの総量に対して一定の比率（例えば P90、P99、P999 の量）でアラートの境界値を計算する必要があります。
* used や query などの監視情報については、成長上限の警告値として2倍以上を設定するか、またはピーク値よりやや高い値を設定することができます。

> 注意：
>
> * 通常、FE と BE を共に監視する必要がありますが、一部の監視情報は FE または BE 固有のものです。
> * クラスタ内のマシンが同種でない場合、複数の監視グループに分ける必要があります。

### ルール設定の参考例

#### 低頻度操作のアラート

低頻度の操作については、失敗が発生した場合（1回以上）に直ちにアラートを出す設定が可能です。複数回の失敗が発生した場合は、より高レベルのアラートを引き起こします。

例えば、テーブル構造の変更などの低頻度の操作に対して、失敗した場合にアラートを出す設定ができます。

#### 未開始タスク

通常、未開始タスクの監視情報は一定期間 `0` です。しかし、このような情報があると、その後に多数の success と failed が発生します。

最初は失敗が1回以上発生した場合にアラートを出す設定にしておき、後で具体的な状況に応じて変更することができます。

#### 変動の大きさ

変動が大きい監視情報については、異なる時間粒度のデータに注意を払う必要があります。なぜなら、大きな粒度のデータでは波のピークと谷が平均化されるからです（将来的には StarRocksManager に sum/min/max/average などの指標集計値を追加する予定です）。通常、15日、3日、12時間、3時間、1時間など、異なる時間範囲のデータを確認する必要があります。また、波動によるアラートを避けるために、より長い監視間隔（例えば3分または5分など）を設定する必要があります。

変動が小さい監視情報については、より短い間隔で設定し、システムがより迅速にアラートを出せるようにすることができます。

監視情報に高いスパイクがある場合、そのスパイクがアラートを出すべきかどうかを判断する必要があります。スパイクが多い場合は、間隔を適切に広げてスパイクを平滑化することができます。スパイクが少ない場合は、通知レベルのアラートを設定することができます。

#### リソースの使用

リソース使用が低い監視情報については、比較的厳しい閾値を設定することができます。例えば、CPU 使用率が低い場合（20%未満）は、`cpu_idle<60%` の時にアラートを出す設定ができます。

リソース使用が高い監視情報については、「一定のリソースを予約する」という方法でアラートを設定することができます。例えば、メモリに対して `mem_avaliable<=20%` の時にアラートを出す設定ができます。

### その他の情報

#### P99 パーセンタイル計算ルール

各ノードは15秒ごとにデータを収集し、対応する数値を計算します。現在の99パーセンタイルは、その15秒間の99パーセンタイルです。QPSが低い場合（例えば10以下）、このパーセンタイルは高い精度を持ちません。また、1分間（15秒間隔の4つ）にわたる4つの値を単純に集計する（sumであれaverageであれ）ことも、数学的に意味をなしません。

上記のルールは、P50やP90の計算にも同様に適用されます。

#### クラスターのエラーモニタリング

クラスター内の予期しないエラー項目を監視し、問題を迅速に発見して解決することで、クラスターを正常な状態に戻す必要があります。一部の監視項目が重要でない場合でも、**重要なエラー項目から一時的に分離することができない**（例えばSQLの構文エラーなど）場合は、一時的に監視を続け、将来的には区別を推進することをお勧めします。

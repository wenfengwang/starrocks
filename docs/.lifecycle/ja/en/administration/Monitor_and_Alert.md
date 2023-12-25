---
displayed_sidebar: English
---

# モニタリングとアラーティング

独自のモニタリングサービスを構築することも、Prometheus + Grafana ソリューションを使用することもできます。StarRocksは、BEおよびFEのHTTPポートに直接リンクして、クラスタからの監視情報を取得するPrometheus互換のインターフェースを提供しています。

## メトリクス

使用可能なメトリクスは以下の通りです。

|メトリック|単位|タイプ|意味|
|---|:---:|:---:|---|
|be_broker_count|カウント|平均|ブローカーの数 |
|be_brpc_endpoint_count|カウント|平均|bRPCのStubCacheの数|
|be_bytes_read_per_second|バイト/秒|平均| BEの読み込み速度 |
|be_bytes_written_per_second|バイト/秒|平均|BEの書き込み速度 |
|be_base_compaction_bytes_per_second|バイト/秒|平均|BEのベースコンパクション速度|
|be_cumulative_compaction_bytes_per_second|バイト/秒|平均|BEの累積コンパクション速度|
|be_base_compaction_rowsets_per_second|rowsets/秒|平均| BEのベースコンパクションrowsets速度|
|be_cumulative_compaction_rowsets_per_second|rowsets/秒|平均| BEの累積コンパクションrowsets速度 |
|be_base_compaction_failed|カウント/秒|平均|BEのベースコンパクション失敗 |
|be_clone_failed| カウント/秒 |平均|BEのクローン失敗 |
|be_create_rollup_failed| カウント/秒 |平均|BEのマテリアライズドビュー作成失敗 |
|be_create_tablet_failed| カウント/秒 |平均|BEのタブレット作成失敗 |
|be_cumulative_compaction_failed| カウント/秒 |平均|BEの累積コンパクション失敗 |
|be_delete_failed| カウント/秒 |平均| BEの削除失敗 |
|be_finish_task_failed| カウント/秒 |平均|BEのタスク完了失敗 |
|be_publish_failed| カウント/秒 |平均| BEのバージョン公開失敗 |
|be_report_tables_failed| カウント/秒 |平均| BEのテーブル報告失敗 |
|be_report_disk_failed| カウント/秒 |平均|BEのディスク報告失敗 |
|be_report_tablet_failed| カウント/秒 |平均|BEのタブレット報告失敗 |
|be_report_task_failed| カウント/秒 |平均|BEのタスク報告失敗 |
|be_schema_change_failed| カウント/秒 |平均|BEのスキーマ変更失敗 |
|be_base_compaction_requests| カウント/秒 |平均|BEのベースコンパクション要求 |
|be_clone_total_requests| カウント/秒 |平均|BEのクローン要求 |
|be_create_rollup_requests| カウント/秒 |平均| BEのマテリアライズドビュー作成要求 |
|be_create_tablet_requests|カウント/秒|平均| BEのタブレット作成要求 |
|be_cumulative_compaction_requests|カウント/秒|平均|BEの累積コンパクション要求 |
|be_delete_requests|カウント/秒|平均| BEの削除要求 |
|be_finish_task_requests|カウント/秒|平均| BEのタスク完了要求 |
|be_publish_requests|カウント/秒|平均| BEのバージョン公開要求 |
|be_report_tablets_requests|カウント/秒|平均|BEのタブレット報告要求 |
|be_report_disk_requests|カウント/秒|平均|BEのディスク報告要求 |
|be_report_tablet_requests|カウント/秒|平均|BEのタブレット報告要求 |
|be_report_task_requests|カウント/秒|平均|BEのタスク報告要求 |
|be_schema_change_requests|カウント/秒|平均|BEのスキーマ変更報告要求 |
|be_storage_migrate_requests|カウント/秒|平均| BEのストレージ移行要求 |
|be_fragment_endpoint_count|カウント|平均|BEのDataStreamの数 |
|be_fragment_request_latency_avg|ミリ秒|平均| フラグメント要求の平均待機時間 |
|be_fragment_requests_per_second|カウント/秒|平均|フラグメント要求の数|
|be_http_request_latency_avg|ミリ秒|平均|HTTP要求の平均遅延 |
|be_http_requests_per_second|カウント/秒|平均|HTTP要求の数|
|be_http_request_send_bytes_per_second|バイト/秒|平均| HTTP要求で送信されたバイト数 |
|fe_connections_per_second|接続/秒|平均| FEの新規接続率 |
|fe_connection_total|接続|累積| FE接続の総数 |
|fe_edit_log_read|操作/秒|平均|FE編集ログの読み取り速度 |
|fe_edit_log_size_bytes|バイト/秒|平均|FE編集ログのサイズ |
|fe_edit_log_write|バイト/秒|平均|FE編集ログの書き込み速度 |
|fe_checkpoint_push_per_second|操作/秒|平均|FEチェックポイントの数 |
|fe_pending_hadoop_load_job|カウント|平均| 保留中のHadoopジョブの数|
|fe_committed_hadoop_load_job|カウント|平均| コミットされたHadoopジョブの数|
|fe_loading_hadoop_load_job|カウント|平均| 読み込み中のHadoopジョブの数|
|fe_finished_hadoop_load_job|カウント|平均| 完了したHadoopジョブの数|
|fe_cancelled_hadoop_load_job|カウント|平均| キャンセルされたHadoopジョブの数|
|fe_pending_insert_load_job|カウント|平均| 保留中の挿入ジョブの数 |
|fe_loading_insert_load_job|カウント|平均| 読み込み中の挿入ジョブの数|
|fe_committed_insert_load_job|カウント|平均| コミットされた挿入ジョブの数|
|fe_finished_insert_load_job|カウント|平均| 完了した挿入ジョブの数|
|fe_cancelled_insert_load_job|カウント|平均| キャンセルされた挿入ジョブの数|
|fe_pending_broker_load_job|カウント|平均| 保留中のブローカージョブの数|
|fe_loading_broker_load_job|カウント|平均| 読み込み中のブローカージョブの数|
|fe_committed_broker_load_job|カウント|平均| コミットされたブローカージョブの数|
|fe_finished_broker_load_job|カウント|平均| 完了したブローカージョブの数|
|fe_cancelled_broker_load_job|カウント|平均| キャンセルされたブローカージョブの数 |
|fe_pending_delete_load_job|カウント|平均| 保留中の削除ジョブの数|
|fe_loading_delete_load_job|カウント|平均| 読み込み中の削除ジョブの数|
|fe_committed_delete_load_job|カウント|平均| コミットされた削除ジョブの数|
|fe_finished_delete_load_job|カウント|平均| 完了した削除ジョブの数|
|fe_cancelled_delete_load_job|カウント|平均| キャンセルされた削除ジョブの数|
|fe_rollup_running_alter_job|カウント|平均| 実行中のロールアップ変更ジョブの数 |
|fe_schema_change_running_job|カウント|平均| 実行中のスキーマ変更ジョブの数 |
|cpu_util| パーセンテージ|平均|CPU使用率 |
|cpu_system | パーセンテージ|平均|cpu_system使用率 |
|cpu_user| パーセンテージ|平均|cpu_user使用率 |
|cpu_idle| パーセンテージ|平均|cpu_idle使用率 |
|cpu_guest| パーセンテージ|平均|cpu_guest使用率 |
|cpu_iowait| パーセンテージ|平均|cpu_iowait使用率 |
|cpu_irq| パーセンテージ|平均|cpu_irq使用率 |
|cpu_nice| パーセンテージ|平均|cpu_nice使用率 |
|cpu_softirq| パーセンテージ|平均|cpu_softirq使用率 |
|cpu_steal| パーセンテージ|平均|cpu_steal使用率 |
|disk_free|バイト|平均| 空きディスク容量 |
|disk_io_svctm|ミリ秒|平均| ディスクIOサービス時間 |
|disk_io_util|パーセンテージ|平均| ディスク使用率 |
|disk_used|バイト|平均| 使用済みディスク容量 |
|starrocks_fe_meta_log_count|カウント|瞬間|チェックポイントのない編集ログの数。`100000`以内の値は妥当とされます。|
|starrocks_fe_query_resource_group|カウント|累積|各リソースグループのクエリ数|
|starrocks_fe_query_resource_group_latency|秒|平均|各リソースグループのクエリ遅延パーセンタイル|
|starrocks_fe_query_resource_group_err|カウント|累積|各リソースグループの誤クエリ数|
|starrocks_be_resource_group_cpu_limit_ratio|パーセンテージ|瞬間|リソースグループのCPU割当比率の瞬間値|

|starrocks_be_resource_group_cpu_use_ratio|パーセンテージ|平均|全リソースグループのCPU時間に対するリソースグループのCPU使用時間の比率|
|starrocks_be_resource_group_mem_limit_bytes|バイト|瞬時|リソースグループのメモリ割り当て上限の瞬時値|
|starrocks_be_resource_group_mem_allocated_bytes|バイト|瞬時|リソースグループのメモリ使用量の瞬時値|
|starrocks_be_pipe_prepare_pool_queue_len|カウント|瞬時|パイプライン準備スレッドプールのタスクキュー長の瞬時値|

## モニタリングアラームのベストプラクティス

モニタリングシステムに関する背景情報:

1. システムは15秒ごとに情報を収集します。
2. 一部の指標は15秒で割り、単位はカウント/秒です。割られていない指標もあり、そのカウントは15秒間隔です。
3. P90、P99などの分位数値は現在、15秒以内で計算されています。より大きな粒度（1分、5分など）で計算する場合は、「平均値」ではなく「特定の値を超えるアラームの数」を使用します。

### 参考情報

1. モニタリングの目的は、異常な状態にのみアラートを出すことであり、正常な状態にはアラートを出さないことです。
2. 異なるクラスタは異なるリソース（例：メモリ、ディスク）と異なる使用状況を持ち、異なる値に設定する必要がありますが、「パーセンテージ」は測定単位として普遍的です。
3. `number of failures`のような指標については、総数の変化を監視し、一定の割合（例えば、P90、P99、P999の量）に基づいてアラームの閾値を計算する必要があります。
4. `2倍以上の値`や`ピーク値を超える値`は、使用量/クエリの増加に対する警告値として一般的に使用されます。

### アラーム設定

#### 低頻度アラーム

1回以上の障害が発生した場合にアラームを発動します。複数の障害がある場合は、さらに高度なアラームを設定します。

頻繁に実行されない操作（例：スキーマ変更）については、「障害発生時にアラーム」が適しています。

#### タスク未開始

モニタリングアラームがオンになると、成功したタスクと失敗したタスクが多くなる可能性があります。`failed > 1`でアラートを設定し、後で修正することができます。

#### 変動

##### 大きな変動

大きな粒度のデータでは、ピークや谷が平均化される可能性があるため、異なる時間粒度のデータに注目する必要があります。通常は、15日間、3日間、12時間、3時間、1時間（異なる時間範囲）を見る必要があります。

変動によるアラームを遮断するためには、監視間隔を少し長くする必要があるかもしれません（例：3分、5分、またはそれ以上）。

##### 小さな変動

問題が発生したときに迅速にアラームを受け取るために、より短い間隔を設定します。

##### 高いスパイク

スパイクにアラームを設定するかどうかは、状況によります。スパイクが多すぎる場合、長い間隔を設定することでスパイクを滑らかにするのに役立つかもしれません。

#### リソース使用

##### 高リソース使用

リソースを少しでも確保するためにアラームを設定します。例えば、メモリアラートを`mem_avaliable<=20%`に設定します。

##### 低リソース使用

「高リソース使用」よりも厳しい値を設定します。例えば、使用率が低いCPU（20%未満）の場合、アラームを`cpu_idle<60%`に設定します。

### 注意事項

通常、FE/BEは一緒に監視されますが、FEまたはBEのみが持つ値もあります。

監視のためにバッチで設定する必要があるマシンもあります。

### 追加情報

#### P99バッチ計算ルール

ノードは15秒ごとにデータを収集し、値を計算します。99パーセンタイルはその15秒間の99パーセンタイルです。QPSが低い場合（例：QPSが10未満）、これらのパーセンタイルはあまり正確ではありません。また、1分間に生成される4つの値（4×15秒）を合計または平均で集計することは意味がありません。

P50、P90なども同様です。

#### クラスタエラーの監視

> クラスタを安定させるためには、望ましくないクラスタエラーをタイムリーに検出して解決する必要があります。エラーが比較的軽微である場合（例：SQL構文エラーなど）、**重要なエラー項目から除外できない**場合は、最初に監視し、後で区別することが推奨されます。

## Prometheus+Grafanaを使用した監視

StarRocksは、データストレージの監視に[Prometheus](https://prometheus.io/)を使用し、結果の視覚化に[Grafana](https://grafana.com/)を使用できます。

### コンポーネント

>このドキュメントでは、PrometheusとGrafanaの実装に基づくStarRocksのビジュアルモニタリングソリューションについて説明します。StarRocksはこれらのコンポーネントの保守や開発を行う責任はありません。PrometheusとGrafanaの詳細については、それぞれの公式ウェブサイトを参照してください。

#### Prometheus

Prometheusは、多次元データモデルと柔軟なクエリ言語を備えた時系列データベースです。監視対象システムからデータをプルするかプッシュしてデータを収集し、それらを時系列データベースに格納します。豊富な多次元データクエリ言語を通じて、さまざまなユーザーのニーズに応えます。

#### Grafana

Grafanaは、さまざまなデータソースをサポートするオープンソースのメトリック分析および可視化システムです。Grafanaは、対応するクエリステートメントを用いてデータソースからデータを取得し、ユーザーがグラフやダッシュボードを作成してデータを可視化することを可能にします。

### 監視アーキテクチャ

![8.10.2-1](../assets/8.10.2-1.png)

PrometheusはFE/BEインターフェースからメトリックをプルし、それらのデータを時系列データベースに格納します。

Grafanaでは、ユーザーはPrometheusをデータソースとして設定し、ダッシュボードをカスタマイズすることができます。

### デプロイメント

#### Prometheus

**1.** [Prometheus公式ウェブサイト](https://prometheus.io/download/)から最新バージョンのPrometheusをダウンロードします。例としてprometheus-2.29.1.linux-amd64バージョンを使用します。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** `vi prometheus.yml`で設定を追加します

```yml
# my global config
global:
  scrape_interval: 15s # Set to 15s here, default is 1m for global acquisition interval
  evaluation_interval: 15s # Set to 15s here, default is 1m for global rule trigger interval

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'StarRocks_Cluster01' # Each cluster is considered a job, and the job name is customizable
    metrics_path: '/metrics' # メトリクスを取得するためのRestful APIを指定します

    static_configs:
      - targets: ['fe_host1:http_port', 'fe_host3:http_port', 'fe_host3:http_port']
        labels:
          group: fe # ここでは3つのフロントエンドを含むFEのグループが設定されています

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # ここでは3つのバックエンドを含むBEのグループが設定されています
  - job_name: 'StarRocks_Cluster02' # Prometheusで複数のStarRocksクラスターを監視できます
metrics_path: '/metrics'

    static_configs:
      - targets: ['fe_host1:http_port', 'fe_host3:http_port', 'fe_host3:http_port']
        labels:
          group: fe

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be
```

**3.** Prometheusを起動する

```bash
nohup ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --log.level="info" &
```

このコマンドはPrometheusをバックグラウンドで実行し、Webポートを9090に指定します。設定が完了すると、Prometheusはデータ収集を開始し、`. /data`ディレクトリに保存します。

**4.** Prometheusへのアクセス

PrometheusはBUI経由でアクセスできます。ブラウザでポート9090を開いてください。`Status -> Targets`に移動して、グループ化されたジョブの監視対象ホストノードを確認します。通常の状況では、すべてのノードが`UP`状態です。ノードのステータスが`UP`でない場合は、まずStarRocksのメトリクス(`http://fe_host:fe_http_port/metrics`または`http://be_host:be_http_port/metrics`)インターフェースにアクセスしてアクセス可能かどうかを確認するか、Prometheusのドキュメントでトラブルシューティングを行ってください。

![8.10.2-6](../assets/8.10.2-6.png)

シンプルなPrometheusが構築され、設定されました。より高度な使用法については、[公式ドキュメント](https://prometheus.io/docs/introduction/overview/)を参照してください。

#### Grafana

**1.** [Grafana公式ウェブサイト](https://grafana.com/grafana/download)から最新バージョンのGrafanaをダウンロードします。例としてgrafana-8.0.6.linux-amd64バージョンを取り上げます。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** `vi ./conf/defaults.ini`で設定を追加します

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

**3.** Grafanaを起動する

```Plain text
nohup ./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

### ダッシュボード

#### ダッシュボードの設定

前のステップで設定したアドレス`http://grafana_host:8000`から、デフォルトのユーザー名、パスワード（admin, admin）を使用してGrafanaにログインします。

**1.** データソースを追加します。

設定パス: `Configuration-->Data sources-->Add data source-->Prometheus`

データソース設定の紹介

![8.10.2-2](../assets/8.10.2-2.png)

* 名前: データソースの名前です。カスタマイズ可能です。例えばstarrocks_monitorとします。
* URL: PrometheusのWebアドレスです。例: `http://prometheus_host:9090`
* アクセス: Server方式を選択します。つまり、Grafanaが配置されているサーバーがPrometheusにアクセスします。
残りのオプションはデフォルトのままです。

下部にある「Save & Test」をクリックし、「Data source is working」と表示されれば、データソースが利用可能であることを意味します。

**2.** ダッシュボードを追加します。

ダッシュボードをダウンロードします。

> **注記**
>
> StarRocks v1.19.0およびv2.4.0のメトリック名が変更されています。StarRocksのバージョンに基づいてダッシュボードテンプレートをダウンロードする必要があります:
>
> * [v1.19.0より前のバージョン用のダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
> * [v1.19.0からv2.4.0までのバージョン用のダッシュボードテンプレート（専用）](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
> * [v2.4.0以降のバージョン用のダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24-new.json)

ダッシュボードテンプレートは定期的に更新されます。

データソースが利用可能であることを確認したら、「+」記号をクリックして新しいダッシュボードを追加します。ここでは上記でダウンロードしたStarRocksダッシュボードテンプレートを使用します。Import -> Upload Json Fileに移動してダウンロードしたjsonファイルを読み込みます。

読み込み後、ダッシュボードに名前を付けることができます。デフォルトの名前はStarRocks Overviewです。次に`starrocks_monitor`をデータソースとして選択します。
「Import」をクリックしてインポートを完了します。するとダッシュボードが表示されます。

#### ダッシュボードの説明

ダッシュボードに説明を追加します。各バージョンの説明を更新します。

**1.** トップバー

![8.10.2-3](../assets/8.10.2-3.png)

左上隅にはダッシュボード名が表示されます。
右上隅には現在の時間範囲が表示されます。ドロップダウンメニューを使用して異なる時間範囲を選択し、ページの更新間隔を指定します。
cluster_name: Prometheusの設定ファイル内の各ジョブの`job_name`で、StarRocksクラスターを表します。クラスターを選択して、その監視情報をチャートで表示できます。

* fe_master: クラスターのリーダーノードです。
* fe_instance: 対応するクラスターのすべてのフロントエンドノードです。選択すると、監視情報がチャートに表示されます。
* be_instance: 対応するクラスターのすべてのバックエンドノードです。選択すると、監視情報がチャートに表示されます。
* interval: 一部のチャートには監視項目に関連する間隔が表示されます。間隔はカスタマイズ可能です（注：15秒間隔では、一部のチャートが表示されない場合があります）。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

Grafanaでは、`Row`は図表のコレクションを意味します。`Row`をクリックすると折りたたむことができます。現在のダッシュボードには以下の`Rows`があります：

* Overview: すべてのStarRocksクラスターの表示。
* Cluster Overview: 選択したクラスターの表示。
* Query Statistic: 選択したクラスターのクエリ監視。
* Jobs: インポートジョブの監視。
* Transaction: トランザクションの監視。
* FE JVM: 選択したフロントエンドのJVM監視。
* BE: 選択したクラスターのバックエンド表示。
* BE Task: 選択したクラスターのバックエンドタスク表示。

**3.** 典型的なチャートは以下の部分に分かれています：

![8.10.2-5](../assets/8.10.2-5.png)

* 左上隅のiアイコンにカーソルを合わせると、チャートの説明が表示されます。
* 下の凡例をクリックすると、特定の項目を表示できます。もう一度クリックすると、すべてが表示されます。
* チャート内をドラッグ&ドロップして時間範囲を選択します。
* 選択したクラスターの名前がタイトルの[]内に表示されます。
* 値は左のY軸または右のY軸に対応しており、凡例の末尾に-rightがあることで区別できます。
* チャート名をクリックして名前を編集します。

### その他

独自のPrometheusシステムで監視データにアクセスする必要がある場合は、以下のインターフェースを通じてアクセスしてください。

* FE: fe_host:fe_http_port/metrics
* BE: be_host:be_web_server_port/metrics

JSONフォーマットが必要な場合は、以下を代わりにアクセスしてください。

* FE: fe_host:fe_http_port/metrics?type=json
* BE: be_host:be_web_server_port/metrics?type=json

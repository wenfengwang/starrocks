---
displayed_sidebar: "Japanese"
---

# モニタリングとアラーム

独自のモニタリングサービスを構築するか、Prometheus + Grafanaのソリューションを使用することができます。StarRocksは、クラスタからのモニタリング情報を取得するために、BEとFEのHTTPポートに直接リンクする、Prometheus互換のインターフェースを提供しています。

## メトリクス

利用可能なメトリクスは以下の通りです：

|メトリクス|単位|タイプ|意味|
|---|:---:|:---:|---|
|be_broker_count|count|average|ブローカーの数 |
|be_brpc_endpoint_count|count|average|bRPCのStubCacheの数|
|be_bytes_read_per_second|bytes/s|average|BEの読み取り速度 |
|be_bytes_written_per_second|bytes/s|average|BEの書き込み速度 |
|be_base_compaction_bytes_per_second|bytes/s|average|BEのベースコンパクション速度|
|be_cumulative_compaction_bytes_per_second|bytes/s|average|BEの累積コンパクション速度|
|be_base_compaction_rowsets_per_second|rowsets/s|average|BEのベースコンパクション速度（rowsets）|
|be_cumulative_compaction_rowsets_per_second|rowsets/s|average|BEの累積コンパクション速度（rowsets）|
|be_base_compaction_failed|count/s|average|BEのベースコンパクションの失敗数 |
|be_clone_failed| count/s |average|BEのクローンの失敗数 |
|be_create_rollup_failed| count/s |average|BEのマテリアライズドビューの作成の失敗数 |
|be_create_tablet_failed| count/s |average|BEのタブレットの作成の失敗数 |
|be_cumulative_compaction_failed| count/s |average|BEの累積コンパクションの失敗数 |
|be_delete_failed| count/s |average|BEの削除の失敗数 |
|be_finish_task_failed| count/s |average|BEのタスクの失敗数 |
|be_publish_failed| count/s |average|BEのバージョンリリースの失敗数 |
|be_report_tables_failed| count/s |average|BEのテーブルレポートの失敗数 |
|be_report_disk_failed| count/s |average|BEのディスクレポートの失敗数 |
|be_report_tablet_failed| count/s |average|BEのタブレットレポートの失敗数 |
|be_report_task_failed| count/s |average|BEのタスクレポートの失敗数 |
|be_schema_change_failed| count/s |average|BEのスキーマ変更の失敗数 |
|be_base_compaction_requests| count/s |average|BEのベースコンパクションのリクエスト数 |
|be_clone_total_requests| count/s |average|BEのクローンのリクエスト数 |
|be_create_rollup_requests| count/s |average|BEのマテリアライズドビューの作成のリクエスト数 |
|be_create_tablet_requests|count/s|average|BEのタブレットの作成のリクエスト数 |
|be_cumulative_compaction_requests|count/s|average|BEの累積コンパクションのリクエスト数 |
|be_delete_requests|count/s|average|BEの削除のリクエスト数 |
|be_finish_task_requests|count/s|average|BEのタスクの完了のリクエスト数 |
|be_publish_requests|count/s|average|BEのバージョンリリースのリクエスト数 |
|be_report_tablets_requests|count/s|average|BEのタブレットレポートのリクエスト数 |
|be_report_disk_requests|count/s|average|BEのディスクレポートのリクエスト数 |
|be_report_tablet_requests|count/s|average|BEのタブレットレポートのリクエスト数 |
|be_report_task_requests|count/s|average|BEのタスクレポートのリクエスト数 |
|be_schema_change_requests|count/s|average|BEのスキーマ変更レポートのリクエスト数 |
|be_storage_migrate_requests|count/s|average|BEのマイグレーションのリクエスト数 |
|be_fragment_endpoint_count|count|average|BEのDataStreamの数 |
|be_fragment_request_latency_avg|ms|average|フラグメントリクエストのレイテンシ |
|be_fragment_requests_per_second|count/s|average|フラグメントリクエストの数|
|be_http_request_latency_avg|ms|average|HTTPリクエストのレイテンシ |
|be_http_requests_per_second|count/s|average|HTTPリクエストの数|
|be_http_request_send_bytes_per_second|bytes/s|average|HTTPリクエストの送信バイト数 |
|fe_connections_per_second|connections/s|average|FEの新しい接続数 |
|fe_connection_total|connections| cumulative | FEの総接続数 |
|fe_edit_log_read|operations/s|average|FEの編集ログの読み取り速度 |
|fe_edit_log_size_bytes|bytes/s|average|FEの編集ログのサイズ |
|fe_edit_log_write|bytes/s|average|FEの編集ログの書き込み速度 |
|fe_checkpoint_push_per_second|operations/s|average|FEのチェックポイントの数 |
|fe_pending_hadoop_load_job|count|average| ハドゥープジョブの保留数 |
|fe_committed_hadoop_load_job|count|average| ハドゥープジョブの完了数 |
|fe_loading_hadoop_load_job|count|average| ハドゥープジョブのロード中数 |
|fe_finished_hadoop_load_job|count|average| ハドゥープジョブの完了数 |
|fe_cancelled_hadoop_load_job|count|average| ハドゥープジョブのキャンセル数 |
|fe_pending_insert_load_job|count|average| インサートジョブの保留数 |
|fe_loading_insert_load_job|count|average| インサートジョブのロード中数 |
|fe_committed_insert_load_job|count|average| インサートジョブの完了数 |
|fe_finished_insert_load_job|count|average| インサートジョブの完了数 |
|fe_cancelled_insert_load_job|count|average| インサートジョブのキャンセル数 |
|fe_pending_broker_load_job|count|average| ブローカージョブの保留数 |
|fe_loading_broker_load_job|count|average| ブローカージョブのロード中数 |
|fe_committed_broker_load_job|count|average| ブローカージョブの完了数 |
|fe_finished_broker_load_job|count|average| ブローカージョブの完了数 |
|fe_cancelled_broker_load_job|count|average| ブローカージョブのキャンセル数 |
|fe_pending_delete_load_job|count|average| 削除ジョブの保留数 |
|fe_loading_delete_load_job|count|average| 削除ジョブのロード中数 |
|fe_committed_delete_load_job|count|average| 削除ジョブの完了数 |
|fe_finished_delete_load_job|count|average| 削除ジョブの完了数 |
|fe_cancelled_delete_load_job|count|average| 削除ジョブのキャンセル数 |
|fe_rollup_running_alter_job|count|average| ロールアップで作成されたジョブの数 |
|fe_schema_change_running_job|count|average| スキーマ変更のジョブの数 |
|cpu_util| percentage|average|CPU使用率 |
|cpu_system | percentage|average|cpu_system使用率 |
|cpu_user| percentage|average|cpu_user使用率 |
|cpu_idle| percentage|average|cpu_idle使用率 |
|cpu_guest| percentage|average|cpu_guest使用率 |
|cpu_iowait| percentage|average|cpu_iowait使用率 |
|cpu_irq| percentage|average|cpu_irq使用率 |
|cpu_nice| percentage|average|cpu_nice使用率 |
|cpu_softirq| percentage|average|cpu_softirq使用率 |
|cpu_steal| percentage|average|cpu_steal使用率 |
|disk_free|bytes|average| 空きディスク容量 |
|disk_io_svctm|ms|average| ディスクIOサービス時間 |
|disk_io_util|percentage|average| ディスク使用率 |
|disk_used|bytes|average| 使用済みディスク容量 |
|starrocks_fe_meta_log_count|count|Instantaneous|チェックポイントのない編集ログの数。`100000`以内の値は合理的とされます。|
|starrocks_fe_query_resource_group|count|cumulative|各リソースグループのクエリ数|
|starrocks_fe_query_resource_group_latency|second|average|各リソースグループのクエリレイテンシのパーセンタイル|
|starrocks_fe_query_resource_group_err|count|cumulative|各リソースグループの不正なクエリ数|
|starrocks_be_resource_group_cpu_limit_ratio|percentage|Instantaneous|リソースグループのCPUクォータ比率の瞬時値|
|starrocks_be_resource_group_cpu_use_ratio|percentage|average|リソースグループのCPU時間の使用率|
|starrocks_be_resource_group_mem_limit_bytes|byte|Instantaneous|リソースグループのメモリクォータの瞬時値|
|starrocks_be_resource_group_mem_allocated_bytes|byte|Instantaneous|リソースグループのメモリ使用量の瞬時値|
|starrocks_be_pipe_prepare_pool_queue_len|count|Instantaneous|パイプラインの準備スレッドプールのタスクキューの瞬時値|

## モニタリングアラームのベストプラクティス

モニタリングシステムの背景情報：

1. システムは15秒ごとに情報を収集します。
2. 一部の指標は15秒で割られ、単位はcount/sです。一部の指標は割られず、countは15秒です。
3. P90、P99などのパーセンタイル値は現在15秒間でカウントされます。より大きな粒度（1分、5分など）で計算する場合は、「ある値より大きいアラームの数」ではなく、「平均値は何ですか」というよりも「何個のアラームがあるか」を使用してください。

### 参考文献

1. モニタリングの目的は、異常な状態のみをアラートすることであり、正常な状態ではアラートしません。
2. 異なるクラスタには異なるリソース（メモリ、ディスクなど）があり、異なる使用状況があり、異なる値に設定する必要があります。ただし、「パーセンテージ」は、測定単位として普遍的です。
3. 「失敗数」などの指標の場合、総数の変化を監視し、一定の比率（たとえば、P90、P99、P999の量）に基づいてアラームの境界値を計算する必要があります。
4. 「2倍以上の値」または「ピークよりも高い値」は、使用/クエリの成長の警告値として一般的に使用できます。

### アラーム設定

#### 低頻度アラーム

1つ以上の失敗が発生した場合にアラームをトリガーします。複数の失敗がある場合は、より高度なアラームを設定します。

頻繁に実行されない操作（スキーマ変更など）の場合、「失敗時にアラームを発生させる」だけで十分です。

#### タスクが開始されていない

モニタリングアラームがオンになると、成功したタスクと失敗したタスクが多数発生する場合があります。`failed > 1`と設定してアラートを発生させ、後で修正します。

#### 変動

##### 大きな変動

大きな粒度のデータにはピークと谷がありますが、これらは平均化される可能性があります。通常、15日、3日、12時間、3時間、1時間（異なる時間範囲に対して）を見る必要があります。

モニタリング間隔は、変動によるアラームをシールドするために、わずかに長くする必要がある場合があります（たとえば、3分、5分、またはそれ以上）。

##### 小さな変動

問題が発生した場合にすばやくアラームを受け取るために、より短い間隔を設定します。

##### 高いスパイク

スパイクをアラートする必要があるかどうかは、状況によります。スパイクが多すぎる場合は、長い間隔を設定することでスパイクを平滑化するのに役立ちます。

#### リソース使用状況

##### リソース使用率が高い

アラームを設定して少しのリソースを確保できます。たとえば、メモリのアラートを`mem_avaliable<=20%`に設定します。

##### リソース使用率が低い

「リソース使用率が高い」よりも厳しい値を設定できます。たとえば、使用率が低いCPU（20%未満）の場合、アラートを`cpu_idle<60%`に設定します。

### 注意事項

通常、FE/BEは一緒に監視されますが、FEまたはBEのみに存在する値がある場合があります。

監視するためにバッチで設定する必要があるマシンがある場合があります。

### 追加情報

#### P99バッチ計算ルール

ノードは15秒ごとにデータを収集し、値を計算します。99パーセンタイルは、その15秒間の99パーセンタイルです。QPSが高くない場合（たとえば、QPSが10未満の場合）、これらのパーセンタイルはあまり正確ではありません。また、1分（4 x 15秒）で生成された4つの値を集計することは意味がありません。

P50、P90なども同様です。

#### クラスタエラーのモニタリング

> クラスタエラーのうち、望ましくないものは、クラスタの安定性を保つために時間内に見つけて解決する必要があります。エラーが重要なエラーアイテムから削除できない場合（たとえば、SQL構文エラーなど）、最初に監視し、後でそれらを区別することをお勧めします。

## Prometheus+Grafanaの使用

StarRocksは、[Prometheus](https://prometheus.io/)を使用してデータストレージを監視し、[Grafana](https://grafana.com/)を使用して結果を視覚化することができます。

### コンポーネント

> このドキュメントでは、PrometheusとGrafanaの実装に基づいたStarRocksの視覚的なモニタリングソリューションについて説明します。StarRocksは、これらのコンポーネントのメンテナンスや開発には関与していません。PrometheusとGrafanaの詳細な情報については、公式ウェブサイトを参照してください。

#### Prometheus

Prometheusは、多次元データモデルと柔軟なクエリ文を持つ時系列データベースです。監視対象のシステムからデータをプルまたはプッシュして、そのデータを時系列データベースに格納します。豊富な多次元データクエリ言語により、さまざまなユーザーのニーズに応えることができます。

#### Grafana

Grafanaは、さまざまなデータソースをサポートするオープンソースのメトリック分析および可視化システムです。Grafanaは、対応するクエリ文を使用してデータソースからデータを取得します。ユーザーは、チャートやダッシュボードを作成してデータを視覚化することができます。

### モニタリングアーキテクチャ

![8.10.2-1](../assets/8.10.2-1.png)

Prometheusは、FE/BEインターフェースからメトリクスを取得し、そのデータを時系列データベースに格納します。

Grafanaでは、ユーザーはPrometheusをデータソースとして構成し、ダッシュボードをカスタマイズすることができます。

### デプロイメント

#### Prometheus

**1.** [Prometheus公式ウェブサイト](https://prometheus.io/download/)から最新バージョンのPrometheusをダウンロードします。ここでは、prometheus-2.29.1.linux-amd64バージョンを使用します。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** `vi prometheus.yml`で設定を追加します。

```yml
# my global config
global:
  scrape_interval: 15s # デフォルトでは1mのグローバル取得間隔ですが、ここでは15sに設定しています
  evaluation_interval: 15s # デフォルトでは1mのグローバルルールトリガー間隔ですが、ここでは15sに設定しています

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'StarRocks_Cluster01' # 各クラスタはジョブと呼ばれ、ジョブ名はカスタマイズ可能です
    metrics_path: '/metrics' # メトリクスを取得するためのRestful APIを指定します

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe # ここではFEのグループが設定されており、3つのフロントエンドが含まれています

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # ここではBEのグループが設定されており、3つのバックエンドが含まれています
  - job_name: 'StarRocks_Cluster02' # Prometheusで複数のStarRocksクラスタを監視できます
    metrics_path: '/metrics'

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be
```

**3.** Prometheusを起動します。

```bash
nohup ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --log.level="info" &
```

このコマンドは、Prometheusをバックグラウンドで実行し、Webポートを9090に指定します。設定が完了すると、Prometheusはデータを収集し、`. /data`ディレクトリにデータを格納し始めます。

**4.** Prometheusへのアクセス

PrometheusはBUI経由でアクセスできます。ブラウザでポート9090を開くだけです。`Status -> Targets`に移動して、監視対象のホストノードをすべてのグループ化されたジョブに表示します。通常、すべてのノードは「UP」である必要があります。ノードのステータスが「UP」でない場合は、まずアクセス可能かどうかを確認するためにStarRocksメトリクス（`http://fe_host:fe_http_port/metrics`または`http://be_host:be_http_port/metrics`）インターフェースにアクセスするか、トラブルシューティングのためにPrometheusのドキュメントを参照してください。

![8.10.2-6](../assets/8.10.2-6.png)

シンプルなPrometheusが構築され、設定されました。より高度な使用法については、[公式ドキュメント](https://prometheus.io/docs/introduction/overview/)を参照してください。

#### Grafana

**1.** [Grafana公式ウェブサイト](https://grafana.com/grafana/download)から最新バージョンのGrafanaをダウンロードします。ここでは、grafana-8.0.6.linux-amd64バージョンを使用します。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** `vi . /conf/defaults.ini`で設定を追加します。

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

**3.** Grafanaを起動します。

```Plain text
nohup ./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

### ダッシュボード

#### ダッシュボードの設定

前の手順で設定したアドレス（`http://grafana_host:8000`）を使用してGrafanaにログインし、デフォルトのユーザー名とパスワード（admin、admin）でログインします。

**1.** データソースを追加します。

設定パス：`Configuration-->Data sources-->Add data source-->Prometheus`

データソースの設定の説明

![8.10.2-2](../assets/8.10.2-2.png)

* Name：データソースの名前。カスタマイズ可能です。たとえば、starrocks_monitor
* URL：PrometheusのWebアドレス。たとえば、`http://prometheus_host:9090`
* Access：サーバーメソッドを選択します。つまり、PrometheusがアクセスするGrafanaが配置されているサーバー。
その他のオプションはデフォルトです。

下部のSave & Testをクリックし、「Data source is working」と表示されれば、データソースが利用可能です。

**2.** ダッシュボードを追加します。

ダッシュボードをダウンロードします。

> **注意**
>
> StarRocks v1.19.0およびv2.4.0のメトリック名が変更されています。StarRocksのバージョンに基づいてダッシュボードテンプレートをダウンロードする必要があります：
>
> * v1.19.0より前のバージョン用のダッシュボードテンプレート
> * v1.19.0からv2.4.0（排他的）までのバージョン用のダッシュボードテンプレート
> * v2.4.0以降のバージョン用のダッシュボードテンプレート

ダッシュボードテンプレートは定期的に更新されます。

データソースが利用可能であることを確認したら、`+`記号をクリックして新しいダッシュボードを追加します。ここでは、上記でダウンロードしたStarRocksのダッシュボードテンプレートを使用します。Import -> Upload Json Fileに移動して、ダウンロードしたjsonファイルをロードします。

ロード後、ダッシュボードに名前を付けることができます。デフォルトの名前はStarRocks Overviewです。次に、データソースとして`starrocks_monitor`を選択します。
`Import`をクリックしてインポートを完了します。その後、ダッシュボードが表示されるはずです。

#### ダッシュボードの説明

ダッシュボードに説明を追加します。各バージョンについて説明を更新します。

**1.** トップバー

![8.10.2-3](../assets/8.10.2-3.png)

左上隅にはダッシュボード名が表示されます。
右上隅には現在の時間範囲が表示されます。ドロップダウンを使用して異なる時間範囲を選択し、ページのリフレッシュ間隔を指定します。
cluster_name：Prometheusの設定ファイルの各ジョブの`job_name`であり、StarRocksクラスタを表します。クラスタを選択し、チャートでそのモニタリング情報を表示できます。

* fe_master：クラスタのリーダーノード。
* fe_instance：対応するクラスタのすべてのフロントエンドノード。チャートでモニタリング情報を表示するために選択します。
* be_instance：対応するクラスタのすべてのバックエンドノード。チャートでモニタリング情報を表示するために選択します。
* interval：一部のチャートは、モニタリングアイテムに関連する間隔を表示します。間隔はカスタマイズ可能です（注意：15秒の間隔では一部のチャートが表示されない場合があります）。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

Grafanaでは、`Row`という概念はダイアグラムのコレクションです。`Row`をクリックすると、`Row`を折りたたむことができます。現在のダッシュボードには次の`Rows`があります：

* Overview：すべてのStarRocksクラスタの表示。
* Cluster Overview：選択したクラスタの表示。
* Query Statistic：選択したクラスタのクエリのモニタリング。
* Jobs：インポートジョブのモニタリング。
* Transaction：トランザクションのモニタリング。
* FE JVM：選択したフロントエンドのJVMのモニタリング。
* BE：選択したクラスタのバックエンドの表示。
* BE Task：選択したクラスタのバックエンドタスクの表示。

**3.** 典型的なチャートは以下のパーツに分かれています。

![8.10.2-5](../assets/8.10.2-5.png)

左上のiアイコンにカーソルを合わせると、チャートの説明が表示されます。
下の凡例をクリックして特定の項目を表示します。再度クリックするとすべて表示されます。
チャート内でドラッグアンドドロップして時間範囲を選択します。
タイトルの[]内に選択したクラスタの名前が表示されます。
値は左のY軸または右のY軸に対応する場合があり、凡例の末尾に-rightが付いていることで区別できます。
チャート名をクリックして名前を編集します。

### その他

独自のPrometheusシステムでモニタリングデータにアクセスする必要がある場合は、次のインターフェースを介してアクセスします。

* FE：fe_host:fe_http_port/metrics
* BE：be_host:be_web_server_port/metrics

JSON形式が必要な場合は、次のインターフェースを介してアクセスします。

* FE：fe_host:fe_http_port/metrics?type=json
* BE：be_host:be_web_server_port/metrics?type=json

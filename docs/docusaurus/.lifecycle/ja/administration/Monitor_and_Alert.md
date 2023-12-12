---
displayed_sidebar: "Japanese"
---

# モニタリングとアラート

独自の監視サービスを構築するか、Prometheus + Grafana ソリューションを使用できます。StarRocks は、クラスターからの監視情報を取得するために BE および FE の HTTP ポートに直接リンクする、Prometheus 互換インターフェースを提供します。

## メトリクス

利用可能なメトリクスは以下の通りです:

|メトリック|単位|タイプ|意味|
|---|:---:|:---:|---|
|be_broker_count|個|平均|ブローカーの数 |
|be_brpc_endpoint_count|個|平均|bRPC の StubCache の数|
|be_bytes_read_per_second|バイト/s|平均| BE の読み取り速度 |
|be_bytes_written_per_second|バイト/s|平均|BE の書き込み速度 |
|be_base_compaction_bytes_per_second|バイト/s|平均|BE のベースコンパクション速度|
|be_cumulative_compaction_bytes_per_second|バイト/s|平均|BE の累積コンパクション速度|
|be_base_compaction_rowsets_per_second|rowsets/s|平均|BE ローセットのベースコンパクション速度|
|be_cumulative_compaction_rowsets_per_second|rowsets/s|平均|BE ローセットの累積コンパクション速度|
|be_base_compaction_failed|個/s|平均|BE のベースコンパクション失敗数 |
|be_clone_failed| 個/s |平均|BE クローン失敗数 |
|be_create_rollup_failed| 個/s |平均|BE マテリアライズドビュー作成失敗数 |
|be_create_tablet_failed| 個/s |平均|BE タブレット作成失敗数 |
|be_cumulative_compaction_failed| 個/s |平均|BE の累積コンパクション失敗数 |
|be_delete_failed| 個/s |平均|BE 削除失敗数 |
|be_finish_task_failed| 個/s |平均|BE タスク終了失敗数 |
|be_publish_failed| 個/s |平均|BE のバージョンリリース失敗数 |
|be_report_tables_failed| 個/s |平均|BE テーブルレポート失敗数 |
|be_report_disk_failed| 個/s |平均|BE ディスクレポート失敗数 |
|be_report_tablet_failed| 個/s |平均|BE タブレットレポート失敗数 |
|be_report_task_failed| 個/s |平均|BE タスクレポート失敗数 |
|be_schema_change_failed| 個/s |平均|BE スキーマ変更失敗数 |
|be_base_compaction_requests| 個/s |平均|BE のベースコンパクションリクエスト数 |
|be_clone_total_requests| 個/s |平均|BE クローンリクエスト数 |
|be_create_rollup_requests| 個/s |平均| BE マテリアライズドビュー作成リクエスト数 |
|be_create_tablet_requests|個/s|平均| BE タブレット作成リクエスト数 |
|be_cumulative_compaction_requests|個/s|平均|BE の累積コンパクションリクエスト数 |
|be_delete_requests|個/s|平均|BE 削除リクエスト数 |
|be_finish_task_requests|個/s|平均|BE タスク終了リクエスト数 |
|be_publish_requests|個/s|平均|BE のバージョンリリースリクエスト数 |
|be_report_tablets_requests|個/s|平均|BE タブレットレポートリクエスト数 |
|be_report_disk_requests|個/s|平均|BE ディスクレポートリクエスト数 |
|be_report_tablet_requests|個/s|平均|BE タブレットレポートリクエスト数 |
|be_report_task_requests|個/s|平均|BE タスクレポートリクエスト数 |
|be_schema_change_requests|個/s|平均|BE スキーマ変更レポートリクエスト数 |
|be_storage_migrate_requests|個/s|平均|BE の移行リクエスト数 |
|be_fragment_endpoint_count|個|平均|BE DataStream の数 |
|be_fragment_request_latency_avg|ms|平均| フラグメントリクエストのレイテンシ|
|be_fragment_requests_per_second|個/s|平均|フラグメントリクエストの数|
|be_http_request_latency_avg|ms|平均|HTTP リクエストのレイテンシ |
|be_http_requests_per_second|個/s|平均|HTTP リクエストの数|
|be_http_request_send_bytes_per_second|バイト/s|平均| HTTP リクエスト送信バイト数 |
|fe_connections_per_second|接続数/s|平均| FE の新規接続率 |
|fe_connection_total|接続数|累積| FE の接続総数 |
|fe_edit_log_read|操作/s|平均|FE の編集ログ読み取り速度 |
|fe_edit_log_size_bytes|バイト/s|平均|FE 編集ログのサイズ |
|fe_edit_log_write|バイト/s|平均|FE 編集ログ書き込み速度 |
|fe_checkpoint_push_per_second|操作/s|平均|FE チェックポイントの数 |
|fe_pending_hadoop_load_job|個|平均| 保留中の Hadoop ジョブ数|
|fe_committed_hadoop_load_job|個|平均| 完了した Hadoop ジョブ数|
|fe_loading_hadoop_load_job|個|平均| 読み込み中の Hadoop ジョブ数|
|fe_finished_hadoop_load_job|個|平均| 完了した Hadoop ジョブ数|
|fe_cancelled_hadoop_load_job|個|平均| キャンセルされた Hadoop ジョブ数|
|fe_pending_insert_load_job|個|平均| 保留中の挿入ジョブ数 |
|fe_loading_insert_load_job|個|平均| 読み込み中の挿入ジョブ数|
|fe_committed_insert_load_job|個|平均| 完了した挿入ジョブ数|
|fe_finished_insert_load_job|個|平均| 完了した挿入ジョブ数|
|fe_cancelled_insert_load_job|個|平均| キャンセルされた挿入ジョブ数|
|fe_pending_broker_load_job|個|平均| 保留中のブローカージョブ数|
|fe_loading_broker_load_job|個|平均| 読み込み中のブローカージョブ数|
|fe_committed_broker_load_job|個|平均| 完了したブローカージョブ数|
|fe_finished_broker_load_job|個|平均| 完了したブローカージョブ数|
|fe_cancelled_broker_load_job|個|平均| キャンセルされたブローカージョブ数 |
|fe_pending_delete_load_job|個|平均| 保留中の削除ジョブ数|
|fe_loading_delete_load_job|個|平均| 読み込み中の削除ジョブ数|
|fe_committed_delete_load_job|個|平均| 完了した削除ジョブ数|
|fe_finished_delete_load_job|個|平均| 完了した削除ジョブ数|
|fe_cancelled_delete_load_job|個|平均| キャンセルされた削除ジョブ数|
|fe_rollup_running_alter_job|個|平均| ロールアップで作成したジョブ数 |
|fe_schema_change_running_job|個|平均| スキーマ変更ジョブ数 |
|cpu_util| パーセンテージ|平均|CPU 使用率 |
|cpu_system | パーセンテージ|平均|cpu_system 使用率 |
|cpu_user| パーセンテージ|平均|cpu_user 使用率 |
|cpu_idle| パーセンテージ|平均|cpu_idle 使用率 |
|cpu_guest| パーセンテージ|平均|cpu_guest 使用率 |
|cpu_iowait| パーセンテージ|平均|cpu_iowait 使用率 |
|cpu_irq| パーセンテージ|平均|cpu_irq 使用率 |
|cpu_nice| パーセンテージ|平均|cpu_nice 使用率 |
|cpu_softirq| パーセンテージ|平均|cpu_softirq 使用率 |
|cpu_steal| パーセンテージ|平均|cpu_steal 使用率 |
|disk_free|バイト|平均| 空きディスク容量 |
|disk_io_svctm|ms|平均| ディスクIOサービス時間 |
|disk_io_util|パーセンテージ|平均| ディスク使用率 |
|disk_used|バイト|平均| 使用済ディスク容量 |
|starrocks_fe_meta_log_count|個|瞬時|チェックポイントなしの編集ログの数。 `100000` の値は合理的と見なされます。|
|starrocks_fe_query_resource_group|個|累積|各リソースグループのクエリ数|
|starrocks_fe_query_resource_group_latency|秒|平均|各リソースグループのクエリ遅延パーセンタイル|
|starrocks_fe_query_resource_group_err|個|累積|各リソースグループの誤ったクエリ数|
|starrocks_be_resource_group_cpu_limit_ratio|パーセンテージ|瞬時|リソースグループのCPU クォータ比の瞬時値|
|starrocks_be_resource_group_cpu_use_ratio|パーセンテージ|平均|リソースグループによるCPU時間の総CPU時間に対する比率|
|starrocks_be_resource_group_mem_limit_bytes|バイト|瞬時|リソースグループのメモリクォータの瞬時値|
|starrocks_be_resource_group_mem_allocated_bytes|バイト|瞬時|リソースグループのメモリ使用量の瞬時値|
|starrocks_be_pipe_prepare_pool_queue_len|個|瞬時|パイプライン準備スレッドプールタスクキューの瞬時値|

## モニタリングアラームのベストプラクティス

モニタリングシステムに関する背景情報:

1. システムは15秒ごとに情報を収集します。
2. 一部の指標は15秒で割られ、単位は個/sです。一部の指標は割られず、カウントは依然として15秒です。
3. P90、P99 およびその他の分位数値は現在15秒でカウントされています。より大きな粒度（1分、5分など）で計算する場合は、「特定の値を超えるアラームの数」を「平均値」ではなく使用します。

### 参考

1. モニタリングの目的は、異常な状態のみにアラートを表示し、正常な状態では表示しないことです。
2. 異なるクラスターには異なるリソース（メモリ、ディスクなど）と、異なる使用状況があり、異なる値に設定する必要があります。ただし、「パーセンテージ」は、測定単位として普遍的です。
3. 「失敗数」のような指標の場合、合計数の変化を監視し、特定の比率に従ってアラームの境界値を計算する必要があります（たとえば、P90、P99、P999 の量に対して）。
4. `2倍以上の値`または`ピーク値を超える値`は、通常使用される成長の警告値として使用できます。
### アラーム設定

#### 低周波アラーム

1つ以上の障害が発生した場合にアラームをトリガーします。複数の障害がある場合には、より高度なアラームを設定します。

頻繁に実行されない操作（例：スキーマ変更）の場合、「障害時にアラーム」が十分です。

#### タスク未開始

監視アラームをオンにすると、多くの成功したタスクや失敗したタスクが発生する可能性があります。`failed > 1` と設定してアラートを出し、後で修正することができます。

#### 変動

##### 大きな変動

大きな時間粒度のデータでは、ピークや谷が平均化される可能性があるため、異なる時間粒度のデータに焦点を当てる必要があります。一般的には、15日、3日、12時間、3時間、および1時間（異なる時間範囲について）を見る必要があります。

変動によるアラームをシールドするために、監視間隔をわずかに長くする必要がある場合があります（例：3分、5分、またはそれ以上）。

##### 小さな変動

問題が発生した際に迅速にアラームを得るために、より短い間隔を設定します。

##### 高いスパイク

スパイクをアラームする必要があるかどうかはそのスパイクに依存します。スパイクがあまり多い場合、より長い間隔を設定することでスパイクを平準化するのに役立ちます。

#### リソース使用

##### リソース使用率が高い

少しのリソースを確保するためにアラームを設定できます。例えば、 メモリのアラートを `mem_avaliable<=20%` と設定します。

##### リソース使用率が低い

「リソース使用率が高い」よりも厳しい値を設定できます。例えば、低いCPU使用率（20%未満）の場合、`cpu_idle<60%` とアラームを設定します。

### 注意

通常、FE/BEは一緒に監視されますが、FEまたはBEのみが持つ値もあります。

監視のためには一括で設定する必要があるマシンがあるかもしれません。

### 追加情報

#### P99バッチの計算規則

ノードは15秒ごとにデータを収集し、値を計算します。99パーセンタイルはその15秒内の99番目のパーセンタイルです。 QPSが高くない場合（たとえば、QPSが10以下の場合）、これらのパーセンタイルはあまり正確ではありません。また、1分間（4x15秒）で生成された4つの値を集計することは意味がありません（合計または平均関数を使用しても）。

P50、P90なども同様です。

#### クラスタエラーの監視

> クラスタエラーは、クラスタを安定させるために時間内に見つけて解決する必要があります。エラーが重要でない場合（SQL構文エラーなど）、**重要でないエラー項目から排除できない場合**、最初に監視し、後でそれらを区別することをお勧めします。

## Prometheus+Grafanaの使用

StarRocksはデータストレージをモニタリングするために[Prometheus](https://prometheus.io/)を使用し、結果を視覚化するために[Grafana](https://grafana.com/)を使用できます。

### コンポーネント

> このドキュメントは、PrometheusとGrafanaの実装に基づいたStarRocksの視覚的監視ソリューションについて説明しています。StarRocksはこれらのコンポーネントの保守または開発に責任を持ちません。PrometheusおよびGrafanaの詳細については、それぞれの公式ウェブサイトを参照してください。

#### Prometheus

Prometheusは多次元データモデルと柔軟なクエリ文を持つ時系列データベースです。監視されたシステムからデータをプルまたはプッシュし、これらのデータを一時的なデータベースに保存します。豊富な多次元データクエリ言語により、さまざまなユーザニーズに対応しています。

#### Grafana

Grafanaはオープンソースのメトリック分析および視覚化システムであり、さまざまなデータソースをサポートしています。Grafanaは対応するクエリ文を使用してデータソースからデータを取得し、ユーザにはデータを視覚化するためのチャートやダッシュボードを作成することができます。

### 監視アーキテクチャ

![8.10.2-1](../assets/8.10.2-1.png)

PrometheusはFE/BEインタフェースからメトリクスをプルし、その後データを一時的なデータベースに保存します。

Grafanaでは、ユーザはPrometheusをデータソースとして構成して、ダッシュボードをカスタマイズすることができます。

### デプロイ

#### Prometheus

**1.** [Prometheus公式ウェブサイト](https://prometheus.io/download/)から最新バージョンのPrometheusをダウンロードします。例として `prometheus-2.29.1.linux-amd64` を取得します。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** `vi prometheus.yml` に設定を追加します。

```yml
# my global config
global:
  scrape_interval: 15s # デフォルトは1mですが、ここでは15秒に設定
  evaluation_interval: 15s # デフォルトは1mですが、ここでは15秒に設定

scrape_configs:
  - job_name: 'StarRocks_Cluster01' # クラスタごとにカスタマイズ可能なジョブ名
    metrics_path: '/metrics' # メトリクスを取得するためのRestful APIを指定

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe # 3つのフロントエンドを含むFEグループがここで設定されています

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # 同様に、3つのバックエンドを含むBEグループがここで設定されています
  - job_name: 'StarRocks_Cluster02' # Prometheusで複数のStarRocksクラスタを監視することができます
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

このコマンドは、Prometheusをバックグラウンドで実行し、webポートを9090に指定しています。設定が完了すると、Prometheusはデータを収集し、`. /data` ディレクトリに保存します。

**4.** Prometheusへのアクセス

PrometheusにはBUIを介してアクセスできます。ブラウザでポート9090を開くだけです。`Status -> Targets` に移動して、グループ化されたすべてのホストノードの監視状態を確認できます。通常の状況では、すべてのノードが `UP` であるはずです。もしノードの状態が `UP` でない場合、まずPrometheusのドキュメントを確認してトラブルシューティングを行うか、StarRocksのメトリクス (`http://fe_host:fe_http_port/metrics` または `http://be_host:be_http_port/metrics`) インタフェースをチェックしてアクセス可能かどうかを確認できます。

![8.10.2-6](../assets/8.10.2-6.png)

Prometheusが構築され、設定されました。より高度な使用法については、[公式ドキュメント](https://prometheus.io/docs/introduction/overview/) を参照してください。

#### Grafana

**1.** [Grafana公式ウェブサイト](https://grafana.com/grafana/download) から最新バージョンのGrafanaをダウンロードします。例として `grafana-8.0.6.linux-amd64` を取得します。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** `vi . /conf/defaults.ini` に設定を追加します。

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

前述のステップで構成したアドレス（`http://grafana_host:8000`）とデフォルトのユーザー名、パスワード（たとえば admin,admin）を使用してGrafanaにログインします。

**1.** データソースを追加します。

構成パス: `Configuration-->Data sources-->Add data source-->Prometheus`

データソースの構成について

![8.10.2-2](../assets/8.10.2-2.png)

* Name: データソースの名前。カスタマイズ可能であり、例えば `starrocks_monitor` とすることができます
* URL: Prometheusのウェブアドレス、例えば `http://prometheus_host:9090`
* Access: サーバーメソッドを選択し、つまりPrometheusがアクセスするサーバーを選択します
その他のオプションはデフォルトです。

最下部の「Save & Test」をクリックし、`データソースが動作している` と表示されれば、データソースが利用可能であることを意味します。

**2.** ダッシュボードを追加します。

ダッシュボードをダウンロードします。

> **注意**
>
> StarRocks v1.19.0およびv2.4.0のメトリック名が変更されています。StarRocksのバージョンに基づいたダッシュボードテンプレートをダウンロードする必要があります。
> * [v1.19.0未満のダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
> * [v1.19.0からv2.4.0（排他）のダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
> * [v2.4.0以降のダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24.json)

ダッシュボードテンプレートは定期的に更新されます。

データソースの使用が確認されたら、新しいダッシュボードを追加するために`+`ボタンをクリックして、上記でダウンロードしたStarRocksダッシュボードテンプレートを使用します。次に、インポート→Jsonファイルをアップロードして、ダウンロードしたjsonファイルを読み込みます。

読み込み後にダッシュボードに名前を付けることができます。デフォルト名はStarRocks概要です。次に、データソースとして`starrocks_monitor`を選択します。
インポートをクリックして、インポートを完了します。その後、ダッシュボードが表示されます。

#### ダッシュボードの説明

ダッシュボードに説明を追加します。各バージョン用に説明を更新します。

**1.** 上部バー

![8.10.2-3](../assets/8.10.2-3.png)

左上隅にはダッシュボード名が表示されます。
右上隅には現在の時間範囲が表示されます。ドロップダウンメニューから別の時間範囲を選択し、ページの更新間隔を指定します。
cluster_name: Prometheus構成ファイルの各ジョブの`job_name`で、StarRocksクラスターを表します。クラスターを選択し、チャートで監視情報を表示できます。

* fe_master: クラスターのリーダーノード。
* fe_instance: 対応するクラスターのすべてのフロントエンドノード。チャートで監視情報を表示するために選択します。
* be_instance: 対応するクラスターのすべてのバックエンドノード。チャートで監視情報を表示するために選択します。
* interval: 一部のチャートは監視アイテムに関連する間隔を表示します。間隔はカスタマイズ可能です(注意: 15秒間隔は一部のチャートを表示しなくすることがあります)。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

Grafanaでは、`Row`という考え方は、図のコレクションです。`Row`をクリックすると、`Row`を折りたたむことができます。現在のダッシュボードには以下の`Row`があります：

* 概要: すべてのStarRocksクラスターの表示。
* クラスター概要: 選択したクラスターの表示。
* クエリ統計: 選択したクラスターのクエリの監視。
* Jobs: インポートジョブの監視。
* Transaction: トランザクションの監視。
* FE JVM: 選択したフロントエンドのJVMの監視。
* BE: 選択したクラスターのバックエンドの表示。
* BE Task: 選択したクラスターのバックエンドタスクの表示。

**3.** 典型的なチャートは以下の部分に分かれています。

![8.10.2-5](../assets/8.10.2-5.png)

* 左上のiアイコンにポインタを合わせると、チャートの説明が表示されます。
* 下の凡例をクリックして特定のアイテムを表示します。再度クリックしてすべてを表示します。
* チャート内でドラッグして時間範囲を選択します。
* タイトルの[]内に選択したクラスターの名前が表示されます。
* 値は左Y軸または右Y軸に対応する場合があります。これは凡例の末尾に-rightが付いていることで区別できます。
* チャート名をクリックして名前を編集します。

### その他

独自のPrometheusシステムで監視データにアクセスする必要がある場合は、以下のインタフェースを介してアクセスします。

* FE: fe_host:fe_http_port/metrics
* BE: be_host:be_web_server_port/metrics

JSON形式が必要な場合は、代わりに以下にアクセスしてください。

* FE: fe_host:fe_http_port/metrics?type=json
* BE: be_host:be_web_server_port/metrics?type=json
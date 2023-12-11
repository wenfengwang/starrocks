---
displayed_sidebar: "Japanese"
---

# モニタリングとアラート

モニタリングサービスを自身で構築するか、Prometheus + Grafana ソリューションを使用することができます。StarRocks は、クラスタからのモニタリング情報を直接取得するために BE と FE の HTTP ポートにリンクする Prometheus 互換のインターフェースを提供します。

## メトリクス

利用可能なメトリクスは次のとおりです。

|メトリック|単位|タイプ|意味|
|---|:---:|:---:|---|
|be_broker_count|count|average|ブローカーの数|
|be_brpc_endpoint_count|count|average|bRPC の StubCache の数|
|be_bytes_read_per_second|bytes/s|average|BE の読み込み速度|
|be_bytes_written_per_second|bytes/s|average|BE の書き込み速度|
|be_base_compaction_bytes_per_second|bytes/s|average|BE のベースコンパクション速度|
|be_cumulative_compaction_bytes_per_second|bytes/s|average|BE の累積コンパクション速度|
...

## モニタリングアラートのベストプラクティス

モニタリングシステムの背景情報:

1. システムは15秒ごとに情報を収集します。
2. 一部の指標は15秒で分割され、単位は count/s です。一部の指標は分割されず、カウントは依然として15秒です。
3. P90、P99 などのパーセンタイル値は現在15秒でカウントされています。より大きな粒度で計算する場合（1分、5分など）、"特定の値を超えるアラームがいくつあるか" を "平均値は何か" の代わりに使用します。

### 参考

1. モニタリングの目的は、異常な状態にのみ警告し、通常の状態には警告しないことです。
2. 異なるクラスタには異なるリソース（メモリ、ディスクなど）と異なる使用状況があり、異なる値を設定する必要がありますが、「パーセンテージ」は計測単位として普遍的です。
3. `失敗の数` などの指標の場合、総数の変化を監視し、一定の比率（たとえば P90、P99、P999 の量に対して）に応じてアラームの境界値を計算する必要があります。
4. 通常の場合よりも `2倍以上の値` や `ピークよりも高い値` は一般的に使用され、使用/クエリの増加の警告値として使用できます。

### アラーム設定

#### 低周波数アラーム

1つ以上の障害が発生した場合にアラームをトリガーします。複数の障害がある場合は、より進化したアラームを設定します。

頻繁に実行されない操作（例: スキーマ変更）の場合、「障害時にアラームを設定」するだけで十分です。

#### タスクが開始されない

監視アラームがオンになると、多くの成功および失敗したタスクが発生する場合があります。`failed > 1`をアラートに設定し、後で変更できます。

#### 変動

##### 大きな変動

異なる時間の粒度を持つデータに焦点を当てる必要があります。大きな粒度のデータでは、ピークと谷が平均化される可能性があります。一般的には、異なる時間範囲について15日、3日、12時間、3時間、1時間を確認する必要があります。

変動によるアラームを遮蔽するために、監視インターバルを若干長くする必要があるかもしれません（例: 3分、5分、またはそれ以上）。

##### 小さな変動

問題が発生した際に迅速にアラームを受け取るため、より短い間隔を設定します。

##### 急激な上昇

急激な上昇をアラートする必要があるかどうかは、状況によります。急激な上昇が多い場合、長いインターバルを設定することで、急激な上昇を抑えるのに役立つかもしれません。

#### リソース使用

##### リソース使用率が高い

少しのリソースを確保するようにアラームを設定できます。例えば、メモリアラートを`mem_available<=20%`と設定します。

##### リソース使用率が低い

「リソース使用率が高い」よりも厳しい値を設定できます。例えば、低い利用率のCPU（20%未満）に対して`cpu_idle<60%`とアラームを設定します。

### 注意事項

通常、FE/BEは一緒に監視されますが、FEまたはBEだけにある値もあります。

モニタリングのためにバッチでセットアップする必要があるマシンもあります。

### 追加情報

#### P99バッチ計算ルール

ノードは15秒ごとにデータを収集し、そのうちの99番目のパーセンタイルを計算します。QPSが高くない場合（例: QPSが10を下回る場合）、これらのパーセンタイルはあまり正確ではありません。また、1分間に生成された4つの値を（合計または平均関数を使用しても）集約することは意味がありません。

P50、P90なども同様です。

#### クラスタエラーのモニタリング

> 安定したクラスタを維持するためには、望ましくないクラスタエラーを見つけて解決する必要があります。エラーが重要ではない場合（例: SQL構文エラーなど）でも**重要なエラー項目から剥がすことができない場合**、まずモニタリングし、後でそれらを区別することを推奨します。

## Prometheus+Grafanaの使用

StarRocksは[Prometheus](https://prometheus.io/)を使用してデータストレージを監視し、結果を可視化するために[Grafana](https://grafana.com/)を使用できます。

### コンポーネント

>このドキュメントは、PrometheusとGrafanaの実装に基づくStarRocksのビジュアルモニタリングソリューションについて説明しています。StarRocksは、これらのコンポーネントの保守や開発には責任を負いません。PrometheusおよびGrafanaの詳細については、公式ウェブサイトを参照してください。

#### Prometheus

Prometheusは、多次元のデータモデルと柔軟なクエリステートメントを持つ一時系列データベースです。監視対象のシステムからデータをプルまたはプッシュして、これらのデータを一時系列データベースに格納します。豊富な多次元データクエリ言語により、さまざまなユーザー要件を満たします。

#### Grafana

Grafanaは、さまざまなデータソースをサポートするオープンソースのメトリクス解析および可視化システムです。Grafanaは対応するクエリステートメントを使用してデータソースからデータを取得します。ユーザーは、チャートやダッシュボードを作成してデータを可視化できます。

### 監視アーキテクチャ

![8.10.2-1](../assets/8.10.2-1.png)

PrometheusはFE/BEインターフェースからメトリクスをプルし、その後データを一時系列データベースに保存します。

Grafanaでは、ユーザーがPrometheusを設定してダッシュボードをカスタマイズできます。

### デプロイ

#### Prometheus

**1.** [Prometheus公式ウェブサイト](https://prometheus.io/download/)から最新バージョンのPrometheusをダウンロードします。ここでは、prometheus-2.29.1.linux-amd64バージョンを例に取ります。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** `vi prometheus.yml`に構成を追加します。

```yml
# my global config
global:
  scrape_interval: 15s # デフォルトでは1分のグローバル取得間隔ですが、ここでは15秒に設定します。
  evaluation_interval: 15s # デフォルトでは1分のグローバルルールトリガ間隔ですが、ここでは15秒に設定します。

scrape_configs:
  # ジョブ名は、この構成からスクレイプされたすべての時系列にラベル`job=<job_name>`として追加されます。
  - job_name: 'StarRocks_Cluster01' # 各クラスタはジョブと呼ばれ、ジョブ名はカスタマイズ可能です
    metrics_path: '/metrics' # メトリクスを取得するRestful APIを指定します

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe # ここでは、3つのフロントエンドを含む「FE」グループが構成されています

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # ここでは、3つのバックエンドを含む「BE」グループが構成されています
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

このコマンドは、Prometheusをバックグラウンドで実行し、ウェブポートを9090に指定します。設定後、Prometheusはデータを収集し、`. /data`ディレクトリにそれを保存します。

**4.** Prometheusへのアクセス

PrometheusはBUI経由でアクセスできます。単純にブラウザでポート9090を開くだけです。`Status -> Targets`に移動して、すべてのグループ化されたジョブのモニタリング対象ノードを見ることができます。通常の状況では、すべてのノードが`UP`であるはずです。ノードの状態が`UP`でない場合は、まず StarRocks メトリクス (`http://fe_host:fe_http_port/metrics`または`http://be_host:be_http_port/metrics`) インターフェースにアクセスしてアクセス可能かどうかを確認するか、トラブルシューティングのためにPrometheusのドキュメントを参照してください。

![8.10.2-6](../assets/8.10.2-6.png)

シンプルなPrometheusが構築および構成されました。さらに高度な使用法については、[公式ドキュメント](https://prometheus.io/docs/introduction/overview/)を参照してください。

#### Grafana

**1.** [Grafana公式ウェブサイト](https://grafana.com/grafana/download)から最新バージョンのGrafanaをダウンロードします。ここでは、grafana-8.0.6.linux-amd64バージョンを例に取ります。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** `. /conf/defaults.ini`に構成を追加します。

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

#### ダッシュボード構成

前述の手順で構成したアドレスを使用して、デフォルトのユーザー名、パスワード（すなわちadmin、admin）でGrafanaにログインします。

**1.** データソースを追加します。

構成パス: `Configuration-->Data sources-->Add data source-->Prometheus`

データソース構成の概要

![8.10.2-2](../assets/8.10.2-2.png)

* 名前: データソースの名前。starrocks_monitorなどをカスタマイズできます
* URL: Prometheusのウェブアドレス、例: `http://prometheus_host:9090`
* アクセス: サーバーメソッドを選択します。すなわち、PrometheusにアクセスするGrafanaが配置されているサーバーです。
その他のオプションはデフォルト値です。

下部の「Save & Test」をクリックし、「データソースが動作している」表示が表示された場合は、データソースが利用可能です。

**2.** ダッシュボードを追加します。

ダッシュボードをダウンロードします。

> **注意**
>
> StarRocks v1.19.0およびv2.4.0のメトリック名が変更されています。StarRocksのバージョンに基づいたダッシュボードテンプレートをダウンロードする必要があります。
> * [v1.19.0未満のバージョン用のダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
> * [v1.19.0からv2.4.0（除外）までのダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
> * [v2.4.0以降用のダッシュボードテンプレート](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24.json)

ダッシュボードテンプレートは定期的に更新されます。

データソースが利用可能であることを確認した後、`+`サインをクリックして新しいダッシュボードを追加し、上記でダウンロードしたStarRocksダッシュボードテンプレートを使用します。 インポート-> Jsonファイルをアップロードして、ダウンロードしたJSONファイルをロードします。

ロード後、ダッシュボードの名前を付けることができます。 デフォルトの名前はStarRocks Overviewです。 次に、データソースとして `starrocks_monitor`を選択します。
インポートをクリックしてインポートを完了します。 その後、ダッシュボードが表示されます。

#### ダッシュボードの説明

ダッシュボードに説明を追加します。 各バージョンの説明を更新します。

**1.** 上部バー

![8.10.2-3](../assets/8.10.2-3.png)

左上隅にはダッシュボードの名前が表示されます。
右上隅には現在の時刻範囲が表示されます。 時間範囲を選択して、ページリフレッシュの間隔を指定するには、ドロップダウンを使用します。
cluster_name: Prometheus構成ファイルの各jobの`job_name` で、StarRocksクラスターを表します。 クラスターを選択して、チャートでそのモニタリング情報を表示できます。

* fe_master: クラスターのリーダーノード。
* fe_instance: 対応するクラスターのすべてのフロントエンドノード。 チャートでモニタリング情報を表示するには選択します。
* be_instance: 対応するクラスターのすべてのバックエンドノード。 チャートでモニタリング情報を表示するには選択します。
* interval: 一部のチャートは監視項目に関連する間隔を表示します。 インターバルはカスタマイズ可能です（注：15秒の間隔は一部のチャートが表示されなくなることがあります）。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

Grafanaでは、`Row`のコンセプトはダイアグラムの集まりです。 `Row`を折りたたむには、それをクリックします。 現在のダッシュボードには、次の`Rows`があります：

* 概要: すべてのStarRocksクラスターの表示。
* クラスター概要: 選択したクラスターの表示。
* クエリ統計: 選択したクラスターのクエリのモニタリング。
* ジョブ: インポートジョブのモニタリング。
* トランザクション: トランザクションのモニタリング。
* FE JVM: 選択したフロントエンドのJVMのモニタリング。
* BE: 選択したクラスターのバックエンドの表示。
* BEタスク: 選択したクラスターのバックエンドタスクの表示。

**3.** 典型的なチャートは次の部分に分かれています。

![8.10.2-5](../assets/8.10.2-5.png)

* 左上隅のiアイコンにポイントを置くと、チャートの説明が表示されます。
* 下の凡例をクリックして特定の項目を表示します。 再度クリックするとすべて表示されます。
* チャート内でドラッグして、時間範囲を選択します。
* 選択したクラスターの名前がタイトルの[]内に表示されます。
* 値は左Y軸または右Y軸に対応する場合があります。 これは凡例の末尾に-rightが付くことで判別できます。
* チャート名をクリックして名前を編集します。

### その他

独自のPrometheusシステムでモニタリングデータにアクセスする必要がある場合は、次のインターフェイスを使用してアクセスします。

* FE: fe_host:fe_http_port/metrics
* BE: be_host:be_web_server_port/metrics

JSON形式が必要な場合は、代わりに次のようにアクセスします。

* FE: fe_host:fe_http_port/metrics?type=json
* BE: be_host:be_web_server_port/metrics?type=json
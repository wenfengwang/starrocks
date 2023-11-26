---
displayed_sidebar: "Japanese"
---

# StarRocksとDatadogの統合

このトピックでは、[Datadog](https://www.datadoghq.com/)というモニタリングおよびセキュリティプラットフォームをStarRocksクラスターと統合する方法について説明します。

## 前提条件

始める前に、インスタンスに以下のソフトウェアがインストールされている必要があります。

- [Datadogエージェント](https://docs.datadoghq.com/getting_started/agent/)
- Python

> **注意**
>
> Datadogエージェントを初めてインストールする際には、Pythonも依存関係としてインストールされます。以下の手順でこのPythonを使用することをおすすめします。

## StarRocksのソースコードの準備

DatadogはまだStarRocksの統合キットを提供していないため、ソースコードを使用して統合する必要があります。

1. ターミナルを起動し、読み書きの両方のアクセス権限があるローカルディレクトリに移動し、次のコマンドを実行してStarRocksのソースコード用の専用ディレクトリを作成します。

    ```shell
    mkdir -p starrocks
    ```

2. 以下のコマンドを使用するか、[GitHub](https://github.com/StarRocks/starrocks/tags)で作成したディレクトリにStarRocksのソースコードパッケージをダウンロードします。

    ```shell
    cd starrocks
    # <starrocks_ver>を実際のStarRocksのバージョン（例：「2.5.2」）に置き換えてください。
    wget https://github.com/StarRocks/starrocks/archive/refs/tags/<starrocks_ver>.tar.gz
    ```

3. パッケージ内のファイルを展開します。

    ```shell
    # <starrocks_ver>を実際のStarRocksのバージョン（例：「2.5.2」）に置き換えてください。
    tar -xzvf <starrocks_ver>.tar.gz --strip-components 1
    ```

## FE統合キットのインストールと設定

1. ソースコードを使用してFEのDatadog統合キットをインストールします。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_fe
    ```

2. FE統合の設定ファイル **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** を作成します。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_fe.d
    sudo cp contrib/datadog-connector/starrocks_fe/datadog_checks/starrocks_fe/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml
    ```

3. FE統合の設定ファイル **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** を編集します。

    いくつかの重要な設定項目の例:

    | **設定** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | fe_metric_url | `http://localhost:8030/metrics` | StarRocks FEメトリクスにアクセスするためのURLです。 |
    | metrics | `- starrocks_fe_*` | FEで監視するメトリクスです。ワイルドカード `*` を使用して設定項目を一致させることができます。 |

## BE統合キットのインストールと設定

1. ソースコードを使用してBEのDatadog統合キットをインストールします。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_be
    ```

2. BE統合の設定ファイル **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** を作成します。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_be.d
    sudo cp contrib/datadog-connector/starrocks_be/datadog_checks/starrocks_be/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml
    ```

3. BE統合の設定ファイル **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** を編集します。

    いくつかの重要な設定項目の例:

    | **設定** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | be_metric_url | `http://localhost:8040/metrics` | StarRocks BEメトリクスにアクセスするためのURLです。 |
    | metrics | `- starrocks_be_*` | BEで監視するメトリクスです。ワイルドカード `*` を使用して設定項目を一致させることができます。 |

## Datadogエージェントの再起動

設定が反映されるように、Datadogエージェントを再起動します。

```shell
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

## 統合の検証

統合を検証する手順については、[Datadogアプリケーション](https://docs.datadoghq.com/getting_started/application/)を参照してください。

## 統合キットのアンインストール

統合キットが不要になった場合は、アンインストールすることができます。

- FE統合キットをアンインストールするには、次のコマンドを実行します:

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-fe
  ```

- BE統合キットをアンインストールするには、次のコマンドを実行します:

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-be
  ```

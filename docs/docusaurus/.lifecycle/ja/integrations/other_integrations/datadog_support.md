---
displayed_sidebar: "Japanese"
---

# StarRocks と Datadog を統合する

このトピックでは、[Datadog](https://www.datadoghq.com/) というモニタリングおよびセキュリティプラットフォームを StarRocks クラスターに統合する方法について説明します。

## 必須事項

始める前に、インスタンスに以下のソフトウェアがインストールされている必要があります。

- [Datadog エージェント](https://docs.datadoghq.com/getting_started/agent/)
- Python

> **注意**
>
> Datadog エージェントを初めてインストールすると、Python も依存関係としてインストールされます。次の手順でこの Python を使用することをお勧めします。

## StarRocksのソースコードの準備

DatadogはまだStarRocks用の統合キットを提供していないため、ソースコードを使用してそれらを統合する必要があります。

1. ターミナルを起動し、読み書きの権限があるローカルディレクトリに移動し、以下のコマンドを実行してStarRocksのソースコード用のディレクトリを作成します。

    ```shell
    mkdir -p starrocks
    ```

2. 以下のコマンドを使用するか、[GitHub](https://github.com/StarRocks/starrocks/tags) から作成したディレクトリにStarRocksのソースコードパッケージをダウンロードします。

    ```shell
    cd starrocks
    # <starrocks_ver> を実際のStarRocksのバージョン、例えば "2.5.2" に置き換えてください。
    wget https://github.com/StarRocks/starrocks/archive/refs/tags/<starrocks_ver>.tar.gz
    ```

3. パッケージ内のファイルを展開します。

    ```shell
    # <starrocks_ver> を実際のStarRocksのバージョン、例えば "2.5.2" に置き換えてください。
    tar -xzvf <starrocks_ver>.tar.gz --strip-components 1
    ```

## FE 統合キットのインストールと構成

1. ソースコードを使用して FE のための Datadog 統合キットをインストールします。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_fe
    ```

2. **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** に FE 統合構成ファイルを作成します。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_fe.d
    sudo cp contrib/datadog-connector/starrocks_fe/datadog_checks/starrocks_fe/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml
    ```

3. **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** を変更します。

    いくつかの重要な構成項目の例:

    | **構成** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | fe_metric_url | `http://localhost:8030/metrics` | StarRocks FE メトリクスにアクセスするための URL。 |
    | metrics | `- starrocks_fe_*` | FE で監視されるメトリクス。ワイルドカード `*` を使用して構成項目に一致させることができます。 |

## BE 統合キットのインストールと構成

1. ソースコードを使用して BE のための Datadog 統合キットをインストールします。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_be
    ```

2. **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** に BE 統合構成ファイルを作成します。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_be.d
    sudo cp contrib/datadog-connector/starrocks_be/datadog_checks/starrocks_be/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml
    ```

3. **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** を変更します。

    いくつかの重要な構成項目の例:

    | **構成** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | be_metric_url | `http://localhost:8040/metrics` | StarRocks BE メトリクスにアクセスするための URL。 |
    | metrics | `- starrocks_be_*` | BE で監視されるメトリクス。ワイルドカード `*` を使用して構成項目に一致させることができます。 |

## Datadog エージェントの再起動

構成が反映されるように、Datadog エージェントを再起動します。

```shell
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

## 統合の確認

統合の確認手順については、[Datadog アプリケーション](https://docs.datadoghq.com/getting_started/application/) を参照してください。

## 統合キットのアンインストール

これらがもはや必要なくなった場合は、統合キットをアンインストールすることができます。

- FE 統合キットをアンインストールするには、以下のコマンドを実行してください:

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-fe
  ```

- BE 統合キットをアンインストールするには、以下のコマンドを実行してください:

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-be
  ```
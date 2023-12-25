---
displayed_sidebar: English
---

# Datadog と StarRocks の統合

このトピックでは、StarRocks クラスターを[Datadog](https://www.datadoghq.com/)（監視およびセキュリティプラットフォーム）と統合する方法について説明します。

## 前提条件

開始する前に、インスタンスに以下のソフトウェアがインストールされている必要があります。

- [Datadog Agent](https://docs.datadoghq.com/getting_started/agent/)
- Python

> **注記**
>
> Datadog Agent を初めてインストールする際には、Python も依存関係としてインストールされます。この Python を次の手順で使用することを推奨します。

## StarRocks のソースコードの準備

Datadog はまだ StarRocks のための統合キットを提供していないため、ソースコードを使用して統合する必要があります。

1. ターミナルを起動し、読み書きの両方のアクセス権を持つローカルディレクトリに移動して、次のコマンドを実行し StarRocks ソースコード専用のディレクトリを作成します。

    ```shell
    mkdir -p starrocks
    ```

2. 次のコマンドを使用するか、[GitHub](https://github.com/StarRocks/starrocks/tags) で作成したディレクトリに StarRocks ソースコードパッケージをダウンロードします。

    ```shell
    cd starrocks
    # <starrocks_ver> を StarRocks の実際のバージョンに置き換えてください。例えば "2.5.2"。
    wget https://github.com/StarRocks/starrocks/archive/refs/tags/<starrocks_ver>.tar.gz
    ```

3. パッケージ内のファイルを展開します。

    ```shell
    # <starrocks_ver> を StarRocks の実際のバージョンに置き換えてください。例えば "2.5.2"。
    tar -xzvf <starrocks_ver>.tar.gz --strip-components 1
    ```

## FE 統合キットのインストールと設定

1. ソースコードを使用して FE 用の Datadog 統合キットをインストールします。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_fe
    ```

2. FE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** を作成します。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_fe.d
    sudo cp contrib/datadog-connector/starrocks_fe/datadog_checks/starrocks_fe/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml
    ```

3. FE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** を編集します。

    いくつかの重要な設定項目の例:

    | **設定** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | fe_metric_url | `http://localhost:8030/metrics` | StarRocks FE メトリクスにアクセスするための URL。 |
    | metrics | `- starrocks_fe_*` | FE で監視するメトリクス。ワイルドカード `*` を使用して設定項目にマッチさせることができます。 |

## BE 統合キットのインストールと設定

1. ソースコードを使用して BE 用の Datadog 統合キットをインストールします。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_be
    ```

2. BE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** を作成します。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_be.d
    sudo cp contrib/datadog-connector/starrocks_be/datadog_checks/starrocks_be/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml
    ```

3. BE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** を編集します。

    いくつかの重要な設定項目の例:

    | **設定** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | be_metric_url | `http://localhost:8040/metrics` | StarRocks BE メトリクスにアクセスするための URL。 |
    | metrics | `- starrocks_be_*` | BE で監視するメトリクス。ワイルドカード `*` を使用して設定項目にマッチさせることができます。 |

## Datadog Agent の再起動

設定を有効にするために Datadog Agent を再起動します。

```shell
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

## 統合の確認

統合を確認する手順については、[Datadog アプリケーション](https://docs.datadoghq.com/getting_started/application/)を参照してください。

## 統合キットのアンインストール

統合キットは不要になったらアンインストールできます。

- FE 統合キットをアンインストールするには、以下のコマンドを実行します。

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-fe
  ```

- BE 統合キットをアンインストールするには、以下のコマンドを実行します。

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-be
  ```

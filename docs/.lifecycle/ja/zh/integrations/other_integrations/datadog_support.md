---
displayed_sidebar: Chinese
---

# StarRocks と Datadog の統合

この文書では、StarRocks クラスターを監視およびセキュリティプラットフォーム [Datadog](https://www.datadoghq.com/) と統合する方法について説明します。

## 前提条件

開始する前に、以下の環境がインストールされていることを確認してください：

- [Datadog Agent](https://docs.datadoghq.com/getting_started/agent/)
- Python

> **説明**
>
> Datadog Agent を初めてインストールする際には、Python も依存関係としてインストールされます。以下の手順でこの Python を使用することをお勧めします。

## StarRocks のソースコードの準備

Datadog プラットフォームは現在 StarRocks の統合スイートを提供していないため、ソースコードを使用してインストールする必要があります。

1. ターミナルを起動し、読み書き権限のあるローカルディレクトリに移動してから、次のコマンドを実行して StarRocks のソースコード用の専用ディレクトリを作成します。

    ```sh
    mkdir -p starrocks
    ```

2. 次のコマンドを使用するか、[GitHub](https://github.com/StarRocks/starrocks/tags) から StarRocks のソースコードパッケージを先に作成したディレクトリにダウンロードします。

    ```sh
    cd starrocks
    # <starrocks_ver> を実際の StarRocks のバージョンに置き換えてください。例えば "2.5.2"。
    wget https://github.com/StarRocks/starrocks/archive/refs/tags/<starrocks_ver>.tar.gz
    ```

3. パッケージ内のファイルを解凍します。

    ```sh
    # <starrocks_ver> を実際の StarRocks のバージョンに置き換えてください。例えば "2.5.2"。
    tar -xzvf <starrocks_ver>.tar.gz --strip-components 1
    ```

## FE 統合スイートのインストールと設定

1. ソースコードを使用して FE に Datadog 統合スイートをインストールします。

    ```sh
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_fe
    ```

2. FE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** を作成します。

    ```sh
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_fe.d
    sudo cp contrib/datadog-connector/starrocks_fe/datadog_checks/starrocks_fe/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml
    ```

3. FE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml** を変更します。

    重要な設定項目の例：

    | **設定項目** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | fe_metric_url | `http://localhost:8030/metrics` | StarRocks FE のメトリクスにアクセスするための URL。 |
    | metrics | `- starrocks_fe_*` | 監視する FE のメトリクス。ワイルドカード `*` を使用して設定項目にマッチさせることができます。 |

## BE 統合スイートのインストールと設定

1. ソースコードを使用して BE に Datadog 統合スイートをインストールします。

    ```sh
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_be
    ```

2. BE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** を作成します。

    ```sh
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_be.d
    sudo cp contrib/datadog-connector/starrocks_be/datadog_checks/starrocks_be/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml
    ```

3. BE 統合設定ファイル **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml** を変更します。

    重要な設定項目の例：

    | **設定項目** | **例** | **説明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | be_metric_url | `http://localhost:8040/metrics` | StarRocks BE のメトリクスにアクセスするための URL。 |
    | metrics | `- starrocks_be_*` | 監視する BE のメトリクス。ワイルドカード `*` を使用して設定項目にマッチさせることができます。 |

## Datadog Agent の再起動

設定を有効にするために Datadog Agent を再起動します。

```sh
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

## 統合の検証

統合を検証するための説明については、[Datadog Application](https://docs.datadoghq.com/getting_started/application/) を参照してください。

## 統合スイートのアンインストール

もはや必要ない場合には、統合スイートをアンインストールすることができます。

- 以下のコマンドを実行して FE 統合ツールキットをアンインストールします：

  ```sh
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-fe
  ```

- 以下のコマンドを実行して BE 統合ツールキットをアンインストールします：

  ```sh
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-be
  ```

---
displayed_sidebar: "Japanese"
---

# MySQLからリアルタイムでデータを同期する

## Flinkジョブがエラーを報告した場合の対処方法

Flinkジョブがエラーを報告し、「Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.」というエラーが発生した場合の可能性として、SMT構成ファイル **config_prod.conf** の `[table-rule.1]` や `[table-rule.2]` など複数のルール設定に必要な構成情報が欠落していることが考えられます。

各ルール設定（例: `[table-rule.1]` や `[table-rule.2]`）が必要なデータベース、テーブル、Flinkコネクタ情報で構成されているかどうかを確認できます。

## Flinkが失敗したタスクを自動的に再起動する方法

Flinkは[チェックポイントメカニズム](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)と[再起動戦略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)を通じて失敗したタスクを自動的に再起動します。

例えば、チェックポイントメカニズムを有効にし、デフォルトの再起動戦略である固定遅延再起動戦略を使用する必要がある場合、以下の情報を設定ファイル **flink-conf.yaml** に構成できます:

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

パラメータの説明:

> **注意**
>
> Flinkドキュメントのより詳細なパラメータの説明については、[チェックポイント](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)を参照してください。

- `execution.checkpointing.interval`: チェックポイントの基本時間間隔。単位: ミリ秒。チェックポイントメカニズムを有効にするには、このパラメータを `0` より大きい値に設定する必要があります。
- `state.backend`: チェックポイントメカニズムが有効になった後、状態はチェックポイントを通じて永続化され、データの損失を防ぎ、復旧後のデータの整合性を確保するため内部的にどのように表現され、どこにどのように永続化されるかを決定するための状態バックエンドを指定します。一般的な値は `filesystem` または `rocksdb` です。状態についての詳細については、[状態バックエンド](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)を参照してください。
- `state.checkpoints.dir`: チェックポイントが書き込まれるディレクトリ。

## Flinkジョブを手動で停止し、停止前の状態に戻す方法

Flinkジョブを停止する際に[セーブポイント](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)を手動でトリガーし（セーブポイントはチェックポイントメカニズムに基づいてストリーミングFlinkジョブの実行状態の一貫したイメージであり、作成されます）、後で指定したセーブポイントからFlinkジョブを復元できます。

1. セーブポイント付きでFlinkジョブを停止します。次のコマンドは、Flinkジョブ `jobId` のために自動的にセーブポイントをトリガーし、Flinkジョブを停止します。さらに、セーブポイントを保存するためのターゲットファイルシステムディレクトリを指定できます。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    パラメータの説明:

    - `jobId`: Flink WebUIでFlinkジョブIDを表示するか、コマンドラインで `flink list -running` を実行することでFlinkジョブIDを表示できます。
    - `targetDirectory`: Flink構成ファイル **flink-conf.yml** でデフォルトディレクトリとして `state.savepoints.dir` を指定できます。セーブポイントがトリガーされると、セーブポイントはこのデフォルトディレクトリに保存され、ディレクトリを指定する必要はありません。

    ```Bash
    state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
    ```

2. 前述のセーブポイントを指定して、Flinkジョブを再送信します。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
    ```
---
displayed_sidebar: "Japanese"
---

# MySQLのデータをリアルタイムで同期する方法

## Flinkジョブがエラーを報告した場合、どうすればよいですか？

Flinkジョブがエラー `Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.` を報告した場合、次の可能性があります。

必要な設定情報が、SMT設定ファイル **config_prod.conf** の `[table-rule.1]` や `[table-rule.2]` などの複数のルールセットで欠落している可能性があります。

各ルールセット（例： `[table-rule.1]` と `[table-rule.2]` ）が、必要なデータベース、テーブル、およびFlinkコネクタの情報で構成されているかどうかを確認できます。

## Flinkが失敗したタスクを自動的に再起動する方法はありますか？

Flinkは、[チェックポイントメカニズム](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)と[再起動戦略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)を通じて、失敗したタスクを自動的に再起動します。

たとえば、チェックポイントメカニズムを有効にし、デフォルトの再起動戦略である固定遅延再起動戦略を使用する場合、次の情報を **flink-conf.yaml** の設定ファイルに設定できます。

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

パラメータの説明：

> **注意**
>
> Flinkドキュメントの詳細なパラメータの説明については、[Checkpointing](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)を参照してください。

- `execution.checkpointing.interval`：チェックポイントの基本時間間隔。単位：ミリ秒。チェックポイントメカニズムを有効にするには、このパラメータを `0` より大きい値に設定する必要があります。
- `state.backend`：状態バックエンドを指定して、状態が内部的にどのように表現され、チェックポイント時にどのようにどこに永続化されるかを決定します。一般的な値は `filesystem` または `rocksdb` です。チェックポイントメカニズムが有効になると、状態はチェックポイント時に永続化され、データの損失を防ぎ、復旧後のデータの整合性を確保します。状態についての詳細は、[State Backends](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)を参照してください。
- `state.checkpoints.dir`：チェックポイントが書き込まれるディレクトリ。

## Flinkジョブを手動で停止し、停止前の状態に復元する方法はありますか？

Flinkジョブを停止するときに[セーブポイント](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)を手動でトリガーし、後で指定したセーブポイントからFlinkジョブを復元することができます（セーブポイントは、チェックポイントメカニズムに基づいて作成されたストリーミングFlinkジョブの実行状態の一貫したイメージです）。

1. セーブポイントを使用してFlinkジョブを停止します。次のコマンドは、Flinkジョブ `jobId` のセーブポイントを自動的にトリガーし、Flinkジョブを停止します。さらに、セーブポイントを保存するためのターゲットファイルシステムディレクトリを指定することもできます。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    パラメータの説明：

    - `jobId`：Flink WebUIからFlinkジョブIDを表示するか、コマンドラインで `flink list -running` を実行してFlinkジョブIDを表示できます。
    - `targetDirectory`：Flinkの設定ファイル **flink-conf.yml** で `state.savepoints.dir` をデフォルトのセーブポイントを保存するディレクトリとして指定できます。セーブポイントがトリガーされると、このデフォルトのディレクトリにセーブポイントが保存され、ディレクトリを指定する必要はありません。

    ```Bash
    state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
    ```

2. 前述のセーブポイントを指定してFlinkジョブを再提出します。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
    ```

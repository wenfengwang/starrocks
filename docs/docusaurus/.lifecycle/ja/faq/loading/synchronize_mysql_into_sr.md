---
displayed_sidebar: "Japanese"
---

# MySQLからリアルタイムでデータを同期する

## Flinkのジョブでエラーが報告された場合はどうすればよいですか？

Flinkのジョブが`Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.`というエラーを報告した場合、可能な原因はSMT構成ファイル**config_prod.conf**の`[table-rule.1]`や`[table-rule.2]`などの複数のルールセットに必要な構成情報が欠落していることです。

各セットのルール、例えば`[table-rule.1]`や`[table-rule.2]`が必要なデータベース、テーブル、およびFlinkコネクタ情報で構成されているかどうかを確認できます。

## Flinkの失敗したタスクを自動的に再起動するにはどうすればよいですか？

Flinkは[checkpointingメカニズム](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)および[再起動戦略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)を通じて失敗したタスクを自動的に再起動します。

例えば、デフォルトの再起動戦略である固定ディレイ再起動戦略を使用する場合、チェックポイントメカニズムを有効にし、以下の情報を**flink-conf.yaml**構成ファイルに設定できます。

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

パラメータの説明：

> **注意**
>
> Flinkのドキュメントでより詳細なパラメータ説明については、[Checkpointing](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)を参照してください。

- `execution.checkpointing.interval`: チェックポイントの基本時間間隔。単位：ミリ秒。チェックポイントメカニズムを有効にするには、このパラメータを`0`より大きい値に設定する必要があります。
- `state.backend`: チェックポイントメカニズムを有効にした後、状態はチェックポイントで永続化され、データの損失を防ぎ、リカバリ後のデータの整合性を保証するために永続化されます。状態についての詳細は、[状態バックエンド](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)を参照してください。
- `state.checkpoints.dir`: チェックポイントが書き込まれるディレクトリ。

## Flinkのジョブを手動で停止し、後で停止前の状態に復元するにはどうすればよいですか？

Flinkのジョブを停止する際に[セーブポイント](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)を手動でトリガーできます（セーブポイントはストリームFlinkジョブの実行状態の一貫したイメージであり、チェックポイントメカニズムに基づいて作成されます）。後で指定したセーブポイントからFlinkジョブを復元することができます。

1. セーブポイントでFlinkジョブを停止します。次のコマンドは、Flinkジョブ`jobId`に対してセーブポイントを自動的にトリガーし、Flinkジョブを停止します。また、保存するセーブポイントのためのターゲットファイルシステムディレクトリを指定できます。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    パラメータの説明：

    - `jobId`: Flink WebUIでFlinkジョブIDを表示したり、コマンドラインで`flink list -running`を実行して表示できます。
    - `targetDirectory`: Flink構成ファイル**flink-conf.yml**でセーブポイントを保存するデフォルトディレクトリとして`state.savepoints.dir`を設定できます。セーブポイントがトリガーされると、このデフォルトディレクトリにセーブポイントが保存され、ディレクトリを指定する必要はありません。

    ```Bash
    state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
    ```

2. 指定したセーブポイントでFlinkジョブを再提出します。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql
    ```

---
displayed_sidebar: Chinese
---

# MySQLのリアルタイム同期に関するStarRocksのよくある質問

## 1. Flink jobの実行エラー

**エラーメッセージ**：`Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.`

**原因分析**：SMT設定ファイル **config_prod.conf** に複数のルール`[table-rule.1]`、`[table-rule.2]`などが設定されていますが、必要な設定情報が不足しています。

**解決方法**：各ルール`[table-rule.1]`、`[table-rule.2]`などにdatabase、table、flink connectorの情報が設定されているか確認してください。

## 2. **Flinkが失敗したタスクを自動的に再起動する方法**

Flinkは[Checkpointingメカニズム](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)と[再起動戦略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)を通じて、失敗したタスクを自動的に再起動します。

例えば、Checkpointingメカニズムを有効にし、デフォルトの再起動戦略である固定遅延再起動戦略を使用する場合、`flink-conf.yaml`設定ファイルで以下のように設定します。

```YAML
# 単位: ms
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

パラメータ説明：

> Flink公式ドキュメントのパラメータ説明は、[Checkpointing](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/dev/datastream/fault-tolerance/checkpointing/)を参照してください。

- `execution.checkpointing.interval`：Checkpointの基本的な時間間隔で、単位はmsです。Checkpointingメカニズムを有効にするには、この値を0より大きい値に設定する必要があります。

- `state.backend`：Checkpointingメカニズムを起動した後、状態はCheckpointと共に永続化され、データの損失を防ぎ、復旧時の一貫性を保証します。内部の状態の保存形式、Checkpoint時の状態の永続化方法、および永続化の場所は、選択したState Backendによって異なります。状態についての詳細は、[State Backends](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)を参照してください。

- `state.checkpoints.dir`：Checkpointデータの保存ディレクトリ。

### 手動でFlink jobを停止し、その後、停止前の状態にFlink jobを復旧する方法

Flink jobを停止する際に手動で[savepoint](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)（savepointはCheckpointingメカニズムに基づいて作成されたストリームジョブの実行状態の一貫性のあるスナップショットです）をトリガーし、後で指定されたsavepointからFlink jobを復旧することができます。

1. Savepointを使用してジョブを停止します。これにより、Flink job `jobId` のsavepointが自動的にトリガーされ、そのジョブが停止します。さらに、Savepointを保存するためのターゲットファイルシステムディレクトリを指定することもできます。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    > 説明
    >
    > - `jobId`：Flink WebUIでFlink job IDを確認するか、コマンドラインで `flink list –running` を実行して確認できます。
    > - `targetDirectory`：また、Flink設定ファイル **flink-conf.yml** の `state.savepoints.dir` でsavepointのデフォルトディレクトリを設定することもできます。savepointをトリガーすると、このディレクトリが使用されてsavepointが保存されるため、ディレクトリを指定する必要はありません。
    >
    > ```Bash
    > state.savepoints.dir: [file://またはhdfs://]/home/user/savepoints_dir
    > ```

2. 停止前の状態にFlink jobを復旧する必要がある場合は、Flink jobを再送信する際にsavepointを指定する必要があります。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
    ```

---
displayed_sidebar: English
---

# MySQLからリアルタイムでデータを同期する

## Flinkジョブがエラーを報告した場合、どうすればいいですか？

Flinkジョブが以下のエラーを報告することがあります：`Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.`

考えられる理由の一つは、SMT設定ファイル**config_prod.conf**内の複数のルールセット、例えば`[table-rule.1]`や`[table-rule.2]`で必要な設定情報が不足していることです。

各ルールセット、例えば`[table-rule.1]`や`[table-rule.2]`が必要なデータベース、テーブル、及びFlinkコネクタ情報を含んでいるかどうかを確認してください。

## 失敗したタスクをFlinkが自動的に再起動するにはどうすればよいですか？

Flinkは[チェックポイントメカニズム](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)と[再起動戦略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)を通じて、失敗したタスクを自動的に再起動します。

例えば、チェックポイントメカニズムを有効にし、デフォルトの再起動戦略である固定遅延再起動戦略を使用する場合、**flink-conf.yaml**設定ファイルに以下の情報を設定します：

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

パラメータの説明：

> **注記**
>
> Flinkドキュメント内のより詳細なパラメータ説明については、[チェックポイント](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)を参照してください。

- `execution.checkpointing.interval`：チェックポイントの基本的な時間間隔です。単位はミリ秒です。チェックポイントメカニズムを有効にするためには、このパラメータを`0`より大きい値に設定する必要があります。
- `state.backend`：状態バックエンドを指定し、状態が内部的にどのように表現され、チェックポイント時にどのように、そしてどこに永続化されるかを決定します。一般的な値には`filesystem`や`rocksdb`があります。チェックポイントメカニズムが有効になると、データ損失を防ぎ、回復後のデータ整合性を保証するために、チェックポイント時に状態が永続化されます。状態に関する詳細は、[ステートバックエンド](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)を参照してください。
- `state.checkpoints.dir`：チェックポイントが書き込まれるディレクトリです。

## Flinkジョブを手動で停止し、後で停止前の状態に復元するにはどうすればよいですか？

Flinkジョブを停止する際に[セーブポイント](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)を手動でトリガーすることができます（セーブポイントは、ストリーミングFlinkジョブの実行状態の一貫したイメージであり、チェックポイントメカニズムに基づいて作成されます）。その後、指定したセーブポイントからFlinkジョブを復元することができます。

1. セーブポイントを使用してFlinkジョブを停止します。以下のコマンドは、Flinkジョブ`jobId`のセーブポイントを自動的にトリガーし、Flinkジョブを停止します。さらに、セーブポイントを格納するためのターゲットファイルシステムディレクトリを指定することもできます。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    パラメータの説明：

    - `jobId`：FlinkジョブIDは、Flink WebUIまたはコマンドラインで`flink list -running`を実行することで確認できます。
    - `targetDirectory`：`state.savepoints.dir`をFlink設定ファイル**flink-conf.yml**でセーブポイントを格納するデフォルトディレクトリとして指定することができます。セーブポイントがトリガーされると、セーブポイントはこのデフォルトディレクトリに保存されるため、ディレクトリを指定する必要はありません。

    ```Bash
    state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
    ```

2. 指定したセーブポイントを使用してFlinkジョブを再提出します。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
    ```

---
displayed_sidebar: English
---

# 实时同步 MySQL 数据

## 如果 Flink 作业报错该怎么办？

Flink 作业报错 `Could not execute SQL statement. Reason: org.apache.flink.table.api.ValidationException: One or more required options are missing.`

可能的原因是 SMT 配置文件 **config_prod.conf** 中多组规则缺少所需的配置信息，例如 `[table-rule.1]` 和 `[table-rule.2]`。

您可以检查每组规则（例如 `[table-rule.1]` 和 `[table-rule.2]`）是否配置了所需的数据库、表和 Flink Connector 信息。

## 如何让 Flink 自动重启失败的任务？

Flink 通过 [checkpointing mechanism](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/) 和 [restart strategy](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/) 自动重启失败的任务。

例如，如果您需要启用 checkpointing mechanism 并使用默认的重启策略，即 fixed delay restart strategy，则可以在配置文件 **flink-conf.yaml** 中配置以下信息：

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

参数说明：

> **注意**
> 在 Flink 文档中查看更详细的参数说明，请参阅 [Checkpointing](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)。

- `execution.checkpointing.interval`：checkpoint 的基本时间间隔。单位：毫秒。要启用 checkpointing mechanism，需要将此参数设置为大于 `0` 的值。
- `state.backend`：指定 state backend 以确定状态在内部如何表示，以及在 checkpoint 时如何以及在何处持久化状态。常见值有 `filesystem` 或 `rocksdb`。启用 checkpointing mechanism 后，状态会在 checkpoint 上持久化，以防止数据丢失并保证恢复后数据的一致性。有关状态的更多信息，请参阅 [State Backends](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)。
- `state.checkpoints.dir`：写入 checkpoint 的目录。

## 如何手动停止 Flink 作业并稍后将其恢复到停止前的状态？

您可以在停止 Flink 作业时手动触发 [savepoint](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)（savepoint 是流式 Flink 作业执行状态的一致镜像，基于 checkpointing mechanism 创建）。稍后，您可以从指定的 savepoint 恢复 Flink 作业。

1. 使用 savepoint 停止 Flink 作业。以下命令自动触发 Flink 作业 `jobId` 的 savepoint 并停止 Flink 作业。此外，您可以指定目标文件系统目录来存储 savepoint。

   ```Bash
   bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
   ```

   参数说明：

   - `jobId`：您可以通过 Flink WebUI 或在命令行中运行 `flink list -running` 查看 Flink 作业 ID。
   - `targetDirectory`：您可以在 Flink 配置文件 **flink-conf.yml** 中指定 `state.savepoints.dir` 作为存储 savepoint 的默认目录。当触发 savepoint 时，savepoint 将存储在该默认目录中，无需指定目录。

   ```Bash
   state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
   ```

2. 使用指定的前述 savepoint 重新提交 Flink 作业。

   ```Bash
   ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
   ```
---
displayed_sidebar: English
---

# 实时同步MySQL数据

## 如果 Flink 作业报告了错误，我该怎么办？

Flink 作业报告了错误 `Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.`

可能的原因是在 SMT 配置文件 **config_prod.conf** 中的多组规则（比如 `[table-rule.1]` 和 `[table-rule.2]`）中缺少了必需的配置信息。

您可以检查每组规则，比如 `[table-rule.1]` 和 `[table-rule.2]`，是否配置了所需的数据库、表和 Flink 连接器信息。

## 如何让 Flink 自动重新启动失败的任务？

Flink 通过[checkpoint 机制](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)和[重启策略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)来自动重新启动失败的任务。

例如，如果您需要启用 checkpoint 机制，并使用默认的重启策略（即固定延迟重启策略），您可以在配置文件 **flink-conf.yaml** 中配置以下信息：

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

参数说明：

> **注意**
>
> 有关 Flink 文档中更详细的参数说明，请参见[Checkpointing](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)。

- `execution.checkpointing.interval`：检查点的基本时间间隔。单位：毫秒。要启用 checkpoint 机制，您需要将此参数设置为大于 `0` 的值。
- `state.backend`：指定状态后端，以确定状态在内部的表示方式，以及状态在检查点时的持久化方式和位置。常用的值有 `filesystem` 或 `rocksdb`。启用 checkpoint 机制后，状态会在检查点时持久化，以防止数据丢失，并确保在恢复后数据的一致性。有关状态的更多信息，请参见[状态后端](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)。
- `state.checkpoints.dir`：检查点写入的目录。

## 如何手动停止 Flink 作业，并在稍后恢复到停止前的状态？

您可以在停止 Flink 作业时手动触发[保存点](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)（保存点是流式 Flink 作业执行状态的一致镜像，基于 checkpoint 机制创建）。稍后，您可以从指定的保存点恢复 Flink 作业。

1. 使用保存点停止 Flink 作业。以下命令会自动触发 Flink 作业 `jobId` 的保存点，并停止 Flink 作业。此外，您还可以指定目标文件系统目录来存储保存点。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    参数说明：

    - `jobId`：您可以通过 Flink WebUI 查看 Flink 作业的 ID，或者在命令行运行 `flink list -running` 来获取。
    - `targetDirectory`：您可以在 Flink 配置文件 **flink-conf.yml** 中指定 `state.savepoints.dir` 作为保存点的默认目录。触发保存点时，保存点将存储在此默认目录中，您无需指定目录。

    ```Bash
    state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
    ```

2. 使用上述保存点重新提交 Flink 作业。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
    ```

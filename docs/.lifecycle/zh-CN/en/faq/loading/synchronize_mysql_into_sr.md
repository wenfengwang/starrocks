---
displayed_sidebar: "Chinese"
---

# 实时同步MySQL中的数据

## 如果一个 Flink 作业报告了错误，我该怎么办？

一个 Flink 作业报告了错误 `Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.`

可能的原因是，在 SMT 配置文件 **config_prod.conf** 中，多组规则（如 `[table-rule.1]` 和 `[table-rule.2]`）缺少了必需的配置信息。

您可以检查每组规则（如 `[table-rule.1]` 和 `[table-rule.2]`）是否配置了必需的数据库、表和 Flink 连接器信息。

## 如何使 Flink 自动重新启动失败的任务？

Flink 通过[检查点机制](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)和[重启策略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)来自动重新启动失败的任务。

例如，如果您需要启用检查点机制并使用默认的重启策略（即固定延迟重启策略），您可以在配置文件 **flink-conf.yaml** 中配置以下信息：

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

参数描述：

> **提示**
> 
> 要了解 Flink 文档中更详细的参数描述，请参阅[检查点机制](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)。

- `execution.checkpointing.interval`：检查点的基础时间间隔。单位：毫秒。要启用检查点机制，您需要将此参数设置为大于 `0` 的值。
- `state.backend`：指定状态后端，以确定状态在内部的表示方式，以及在检查点时状态如何以及何处持久化。常见的值为 `filesystem` 或 `rocksdb`。启用检查点机制后，状态在检查点时会被持久化，以防止数据丢失，并在恢复后确保数据一致性。有关状态的更多信息，请参阅[状态后端](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)。
- `state.checkpoints.dir`：用于写入检查点的目录。

## 如何手动停止 Flink 作业，并稍后将其恢复到停止前的状态？

您可以在停止 Flink 作业时手动触发[保存点](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)（保存点是流式 Flink 作业执行状态的一致图像，并根据检查点机制创建）。稍后，您可以从指定的保存点恢复 Flink 作业。

1. 带有保存点停止 Flink 作业。以下命令会自动为 Flink 作业 `jobId` 触发一个保存点，并停止 Flink 作业。另外，您可以指定一个目标文件系统目录来存储保存点。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    参数描述：

    - `jobId`：您可以从 Flink WebUI 或通过在命令行上运行 `flink list -running` 来查看 Flink 作业 ID。
    - `targetDirectory`：您可以将 `state.savepoints.dir` 指定为默认目录，用于在 Flink 配置文件 **flink-conf.yml** 中存储保存点。触发保存点时，保存点将存储在此默认目录中，而您无需指定目录。

    ```Bash
    state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
    ```

2. 使用之前指定的保存点重新提交 Flink 作业。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
    ```
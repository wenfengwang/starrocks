---
displayed_sidebar: "中文"
---

# MySQL 实时同步至 StarRocks 常见问题

## 1. 执行 Flink job 报错

**报错提示**：`Could not execute SQL statement. Reason:org.apache.flink.table.api.ValidationException: One or more required options are missing.`

**原因分析**：在SMT配置文件 **config_prod.conf** 中设置了多组规则`[table-rule.1]`、`[table-rule.2]`等但是缺失必要的配置信息。

**解决方式**：检查是否给每组规则`[table-rule.1]`、`[table-rule.2]`等配置database，table和 flink connector 信息。

## 2. **Flink 如何自动重启失败的 Task**

Flink 通过[Checkpointing 机制](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)和[重启策略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)，自动重启失败的 Task。

例如，您需要启用Checkpointing机制，并且使用默认的重启策略，即固定延时重启策略，则可以在配置文件`flink-conf.yaml`进行如下配置。

```YAML
# unit: ms
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

参数说明：

> Flink 官网文档的参数说明，请参见[Checkpointing](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/dev/datastream/fault-tolerance/checkpointing/)。

- `execution.checkpointing.interval`：Checkpoint 的基本时间间隔，单位为 ms。如果需要启用Checkpointing机制，则您需要设置该值为大于 0。

- `state.backend`：启动Checkpointing机制后，状态会随着CheckPoint而持久化，以防止数据丢失、保障恢复时的一致性。状态内部的存储格式、状态在CheckPoint时如何持久化以及持久化在哪里均取决于选择的State Backend。状态更多介绍，请参见[State Backends](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)。

- `state.checkpoints.dir`：Checkpoint 数据存储目录。

### 如何手动停止的 Flink job，并且后续恢复 Flink job 至停止前的状态

可以在停止 Flink job 时手动触发[savepoint](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)（savepoint是依据Checkpointing机制所创建的流作业执行状态的一致镜像），后续可以从指定savepoint中恢复flinkjob。

1. 使用Savepoint停止作业。这将自动触发Flink job `jobId`的savepoint，并停止该job。此外，你可以指定一个目标文件系统目录来存储Savepoint。

    ```Bash
    bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
    ```

    > 说明
    >
    > - `jobId`: 您可以通过Flink WebUI查看Flink job ID，或者在命令行执行`flink list –running`进行查看。
    > - `targetDirectory`: 您也可以在Flink配置文件 **flink-conf.yml** 中`state.savepoints.dir`配置savepoint的默认目录。 触发savepoint时，将使用此目录来存储savepoint，无需指定目录。
    >
    > ```Bash
    > state.savepoints.dir: [file://或hdfs://]/home/user/savepoints_dir
    > ```

2. 如果需要恢复Flink job至停止前的状态，则您需要在重新提交Flink job时指定savepoint。

    ```Bash
    ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
    ```
---
displayed_sidebar: English
---

# 实时同步MySQL数据

## 如果Flink作业报错该怎么办？

Flink作业报错：“无法执行SQL语句。原因：org.apache.flink.table.api.ValidationException：缺少一个或多个必需的选项。”

可能的原因是SMT配置文件**config_prod.conf**中的多组规则缺失了必要的配置信息，如`[table-rule.1]`和`[table-rule.2]`。

您可以检查每组规则（例如[table-rule.1]和[table-rule.2]）是否配置有所需的数据库、表以及Flink连接器的信息。

## 如何让Flink自动重启失败的任务？

Flink通过[检查点机制](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)和[重启策略](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/ops/state/task_failure_recovery/)来自动重启失败的任务。

例如，如果您想启用检查点机制并使用默认的重启策略——固定延迟重启策略，您可以在配置文件**flink-conf.yaml**中设置以下信息：

```Bash
execution.checkpointing.interval: 300000
state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
```

参数描述：

> **注意**
> 在Flink文档中，了解更详细的参数描述，请参阅[检查点](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/fault-tolerance/checkpointing/)。

- execution.checkpointing.interval：检查点的基准时间间隔。单位：毫秒。要启用检查点机制，您需要将此参数设置为大于0的数值。
- `state.backend`：指定状态后端，用以确定状态在内部如何表示，以及在执行检查点时它将如何以及在哪里被持久化。常见的值包括`filesystem`或`rocksdb`。启用检查点机制后，状态会在检查点时被持久化，以防止数据丢失并确保数据在恢复后的一致性。更多信息，请参阅[状态后端](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/state_backends/)。
- state.checkpoints.dir：检查点写入的目录。

## 如何手动停止一个Flink作业，并在之后将其恢复到停止前的状态？

您可以在停止Flink作业时手动触发一个[保存点](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/state/savepoints/)（保存点是一个流式Flink作业执行状态的一致性镜像，基于检查点机制创建的）。之后，您可以从指定的保存点恢复Flink作业。

1. 带保存点停止Flink作业。以下命令会为Flink作业jobId自动触发一个保存点并停止该作业。另外，您还可以指定一个目标文件系统目录来存储保存点。

   ```Bash
   bin/flink stop --type [native/canonical] --savepointPath [:targetDirectory] :jobId
   ```

   参数描述：

   -  jobId：您可以通过Flink WebUI或在命令行执行flink list -running来查看Flink作业ID。
   -  `targetDirectory`: 您可以指定`state.savepoints.dir`作为**flink-conf.yml**中存储保存点的默认目录。当触发保存点时，保存点将被存储在这个默认目录，而无需指定一个目录。

   ```Bash
   state.savepoints.dir: [file:// or hdfs://]/home/user/savepoints_dir
   ```

2. 使用之前指定的保存点重新提交Flink作业。

   ```Bash
   ./flink run -c com.starrocks.connector.flink.tools.ExecuteSQL -s savepoints_dir/savepoints-xxxxxxxx flink-connector-starrocks-xxxx.jar -f flink-create.all.sql 
   ```

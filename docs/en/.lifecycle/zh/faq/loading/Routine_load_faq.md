---
displayed_sidebar: English
---

# 常规负载

## 如何提高加载性能？

**方法 1：增加实际加载任务的并行度**，通过将一个加载作业拆分为尽可能多的并行加载任务。

> **注意**
> 此方法可能会消耗更多的 CPU 资源，并导致过多的 tablet 版本。

实际加载任务的并行度由以下几个参数组成的公式决定，其上限是存活的 BE 节点数量或待消费的分区数。

```Plaintext
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

参数描述：

- `alive_be_number`：存活的 BE 节点数量。
- `partition_number`：待消费的分区数。
- `desired_concurrent_number`：Routine Load 作业期望的加载任务并行度。默认值为 `3`。您可以为此参数设置更高的值来提高实际加载任务的并行度。
  - 如果您尚未创建 Routine Load 作业，需要在使用 [CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 创建 Routine Load 作业时设置此参数。
  - 如果您已经创建了 Routine Load 作业，需要使用 [ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) 来修改此参数。
- `max_routine_load_task_concurrent_num`：Routine Load 作业的默认最大任务并行度，默认值为 `5`。这是一个 FE 动态参数。更多信息和配置方法，请参见[参数配置](../../administration/FE_configuration.md#loading-and-unloading)。

因此，当待消费的分区数和存活的 BE 节点数大于其他两个参数时，您可以通过增加 `desired_concurrent_number` 和 `max_routine_load_task_concurrent_num` 参数的值来提高实际加载任务的并行度。

例如，待消费的分区数为 `7`，存活的 BE 节点数为 `5`，`max_routine_load_task_concurrent_num` 为默认值 `5`。此时，如果您需要将加载任务并行度提升至上限，应将 `desired_concurrent_number` 设置为 `5`（默认值为 `3`）。那么，实际任务并行度 `min(5,7,5,5)` 计算得出为 `5`。

有关更多参数描述，请参见 [CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

**方法 2：增加 Routine Load 任务从一个或多个分区消费的数据量。**

> **注意**
> 此方法可能会导致数据加载延迟。

Routine Load 任务可以消费的消息数量上限由参数 `max_routine_load_batch_size`（即加载任务可以消费的最大消息数量）或参数 `routine_load_task_consume_second`（即消息消费的最大持续时间）决定。一旦加载任务消费了满足任一条件的足够数据，消费就完成了。这两个参数是 FE 动态参数。更多信息和配置方法，请参见[参数配置](../../administration/FE_configuration.md#loading-and-unloading)。

您可以通过查看 **be/log/be.INFO** 日志来分析哪个参数决定了加载任务消费数据量的上限。通过增加该参数，您可以提高加载任务消费的数据量。

```Plaintext
I0325 20:27:50.410579 15259 data_consumer_group.cpp:131] consumer group done: 41448fb1a0ca59ad-30e34dabfa7e47a0. consume time(ms)=3261, received rows=179190, received bytes=9855450, eos: 1, left_time: -261, left_bytes: 514432550, blocking get time(us): 3065086, blocking put time(us): 24855
```

通常情况下，日志中的 `left_bytes` 字段大于等于 `0`，表明加载任务在 `routine_load_task_consume_second` 内消费的数据量未超过 `max_routine_load_batch_size`。这意味着一批定时的加载任务能够消费 Kafka 中的所有数据，无需延迟消费。在这种情况下，您可以为 `routine_load_task_consume_second` 设置更大的值，以增加加载任务从一个或多个分区消费的数据量。

如果 `left_bytes` 字段小于 `0`，则表示加载任务在 `routine_load_task_consume_second` 内消费的数据量已达到 `max_routine_load_batch_size`。这意味着每次 Kafka 的数据都能填满一批计划中的加载任务。因此，很可能 Kafka 中还有未被消费的剩余数据，导致消费延迟。在这种情况下，您可以为 `max_routine_load_batch_size` 设置更大的值。

## 如果 SHOW ROUTINE LOAD 的结果显示加载作业处于 `PAUSED` 状态，该怎么办？

- 检查 `ReasonOfStateChanged` 字段，它报告了错误消息 `Broker: Offset out of range`。

  **原因分析：**加载作业的 consumer offset 在 Kafka 分区中不存在。

  **解决方案：**您可以执行 [SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 并检查参数 `Progress` 中加载作业的最新 consumer offset。然后，您可以验证 Kafka 分区中是否存在相应的消息。如果不存在，可能是因为：

  - 创建加载作业时指定的消费者偏移量是未来的偏移量。
  - Kafka 分区中指定消费者偏移处的消息在加载作业消费之前已被删除。建议根据加载速度设置合理的 Kafka 日志清理策略和参数，例如 `log.retention.hours` 和 `log.retention.bytes`。

- 检查 `ReasonOfStateChanged` 字段，它没有报告错误消息 `Broker: Offset out of range`。

  **原因分析：**加载任务的错误行数超过了阈值 `max_error_number`。

  **解决方案：**您可以使用 `ReasonOfStateChanged` 和 `ErrorLogUrls` 字段中的错误消息来排查并修复问题。

-   如果是由于数据源中的数据格式不正确导致的，则需要检查数据格式并修复问题。成功修复问题后，您可以使用 [RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 恢复暂停的加载作业。

-   如果是因为 StarRocks 无法解析数据源中的数据格式，则需要调整阈值 `max_error_number`。您可以先执行 [SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 查看 `max_error_number` 的值，然后使用 [ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) 增大阈值。修改阈值后，您可以使用 [RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) 来恢复暂停的加载作业。

## 如果 SHOW ROUTINE LOAD 的结果显示加载作业处于 `CANCELLED` 状态，该怎么办？

**原因分析：**加载作业在加载过程中遇到异常，例如表被删除。

**解决方案：**在排查和修复问题时，您可以参考 `ReasonOfStateChanged` 和 `ErrorLogUrls` 字段中的错误消息。然而，修复问题后，您无法恢复已取消的加载作业。

## Routine Load 在从 Kafka 消费并写入 StarRocks 时能否保证一致性语义？

Routine Load 保证了精确一次（exactly-once）语义。

每个加载任务都是一个独立的事务。如果事务执行过程中出现错误，则事务将被中止，FE 不会更新加载任务相关分区的消费进度。当 FE 下次从任务队列调度加载任务时，加载任务会从分区最后保存的消费位置开始发送消费请求，从而确保了精确一次语义。
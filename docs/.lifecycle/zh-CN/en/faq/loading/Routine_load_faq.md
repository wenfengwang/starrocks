---
displayed_sidebar: "Chinese"
---

# 常规加载

## 如何提高加载性能？

**方法 1：增加实际加载任务的并行度**，将加载作业拆分为尽可能多的并行加载任务。

> **提示**
>
> 此方法可能会消耗更多的 CPU 资源，并导致太多的表版本。

实际加载任务的并行性由以下公式确定，由多个参数组成，上限为 BE 节点存活数或要消耗的分区数。

```Plaintext
min(存活的 BE 节点数, 要消耗的分区数, 期望的并行数, 最大常规加载任务并行数)
```

参数说明：

- `存活的 BE 节点数`：存活的 BE 节点数量。
- `要消耗的分区数`：要消耗的分区数量。
- `期望的并行数`：常规加载作业的期望加载任务并行性。默认值为 `3`。您可以为此参数设置更高的值，以增加实际加载任务的并行性。
  - 如果尚未创建常规加载作业，则在使用 [CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 创建常规加载作业时，需要设置此参数。
  - 如果已经创建了常规加载作业，则需要使用 [ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) 修改此参数。
- `最大常规加载任务并行数`：常规加载作业的默认最大任务并行性。默认值为 `5`。此参数是一个 FE 动态参数。有关更多信息和配置方法，请参见[参数配置](../../administration/Configuration.md#loading-and-unloading)。

因此，当要消耗的分区数和存活的 BE 节点数大于另外两个参数时，您可以增加 `期望的并行数` 和 `最大常规加载任务并行数` 参数的值，以增加实际加载任务的并行性。

例如，要消耗的分区数为 `7`，存活的 BE 节点数为 `5`，且`最大常规加载任务并行数` 为默认值 `5`。此时，如果需要将加载任务的并行度增加到上限，需要将 `期望的并行数` 设置为 `5`（默认值为 `3`）。然后，计算实际任务并行性`min(5,7,5,5)`，得到 `5`。

有关更多参数说明，请参见 [CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

**方法 2：增加常规加载任务从一个或多个分区消耗的数据量。**

> **提示**
>
> 此方法可能会导致数据加载延迟。

常规加载任务可以消耗的消息数量上限由参数 `max_routine_load_batch_size` 或参数 `routine_load_task_consume_second` 决定。`max_routine_load_batch_size` 表示加载任务可以消耗的最大消息数量，`routine_load_task_consume_second` 表示消息消耗的最大持续时间。一旦加载任务消耗的数据量达到两者之一的要求，消耗完成。这两个参数是 FE 动态参数。有关更多信息和配置方法，请参见[参数配置](../../administration/Configuration.md#loading-and-unloading)。

您可以通过查看 **be/log/be.INFO** 中的日志来分析哪个参数确定了加载任务消耗的数据量上限。通过增加该参数，您可以增加加载任务消耗的数据量。

```Plaintext
I0325 20:27:50.410579 15259 data_consumer_group.cpp:131] consumer group done: 41448fb1a0ca59ad-30e34dabfa7e47a0. consume time(ms)=3261, received rows=179190, received bytes=9855450, eos: 1, left_time: -261, left_bytes: 514432550, blocking get time(us): 3065086, blocking put time(us): 24855
```

通常，日志中的 `left_bytes` 字段大于或等于 `0`，表示加载任务消耗的数据量在`routine_load_task_consume_second` 内未超过 `max_routine_load_batch_size`，这意味着一批已安排的加载任务可以完全消耗来自 Kafka 的所有数据，而不会出现消耗延迟。在这种情况下，您可以设置 `routine_load_task_consume_second` 的更大值，以增加常规加载任务从一个或多个分区消耗的数据量。

如果 `left_bytes` 字段小于 `0`，表示加载任务消耗的数据量在 `routine_load_task_consume_second` 内达到 `max_routine_load_batch_size`。每次来自 Kafka 的数据填充已安排的加载任务的批处理。因此，极有可能有未消耗的数据留存在 Kafka 中，导致消耗延迟。在这种情况下，您可以设置 `max_routine_load_batch_size` 的更大值。

## SHOW ROUTINE LOAD 的结果显示加载作业处于`PAUSED`状态时应该如何处理？

- 检查字段 `ReasonOfStateChanged`，如果报告错误消息`Broker: Offset out of range`。

  **原因分析：** 加载作业的消费者偏移量在 Kafka 分区中不存在。

  **解决方法：** 您可以执行[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)，并在参数 `Progress` 中检查加载作业的最新消费者偏移量。然后，您可以验证相应的消息是否存在于 Kafka 分区中。如果不存在，可能是因为

  - 创建加载作业时指定的消费者偏移量是未来的偏移量。
  - 在被加载作业消耗之前，Kafka 分区中指定消费者偏移量的消息已被删除。建议根据加载速度设置合理的 Kafka 日志清理策略和参数，例如 `log.retention.hours` 和 `log.retention.bytes`。

- 检查字段 `ReasonOfStateChanged`，如果没有报告错误消息`Broker: Offset out of range`。

  **原因分析：** 加载任务中的错误行数超过了阈值`max_error_number`。

  **解决方法：** 您可以通过使用字段`ReasonOfStateChanged`和`ErrorLogUrls`中的错误消息来进行故障排除和修复。

  - 如果是因为数据源中有不正确的数据格式而导致的问题，您需要检查数据格式并解决问题。成功修复问题后，可以使用[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)来恢复暂停的加载作业。

  - 如果是因为 StarRocks 无法解析数据源中的数据格式，您需要调整阈值`max_error_number`。您可以首先执行[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) 来查看 `max_error_number` 的值，然后使用[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) 来增加该阈值。在修改阈值后，可以使用[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)来恢复暂停的加载作业。

## SHOW ROUTINE LOAD 的结果显示加载作业处于`CANCELLED`状态时应该如何处理？

  **原因分析：** 加载作业在加载过程中遇到异常，例如表已删除。

  **解决方法：** 在故障排除和修复问题时，您可以参考字段 `ReasonOfStateChanged` 和 `ErrorLogUrls` 中的错误消息。但是，在修复问题后，无法恢复已取消的加载作业。

## 常规加载能否保证从 Kafka 消费并写入 StarRocks 时的一致语义？

   常规加载保证精确一次语义。

   每个加载任务都是一个独立的事务。如果事务执行过程中出现错误，则该事务会被中止，FE 不会更新相关加载任务的分区消费进度。当 FE 下次从任务队列调度加载任务时，加载任务将从分区的上次保存的消费位置发送消费请求，从而确保精确一次语义。
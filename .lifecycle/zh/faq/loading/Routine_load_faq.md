---
displayed_sidebar: English
---

# 常规加载

## 如何提升加载性能？

**方法1：**通过将加载作业分割成尽可能多的并行加载任务来提高实际的加载任务并行度。

> **注意**
> 这种方法可能会消耗更多的CPU资源，并可能导致过多的平板电脑版本。

实际加载任务的并行度由以下几个参数组成的公式决定，其上限为存活的BE节点数量或待消费的分区数。

```Plaintext
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

参数描述：

- alive_be_number：存活的BE节点数量。
- partition_number：待消费的分区数。
- desired_concurrent_number：常规加载作业期望的加载任务并行度。默认值为3。您可以为此参数设置一个更高的值，以增加实际加载任务的并行度。
  - 如果您还没有创建常规加载作业，需要在使用[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)创建常规加载作业时设置此参数。
  - 如果您已经创建了常规加载作业，则需要使用[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)来修改此参数。
- `max_routine_load_task_concurrent_num`：常规加载作业的默认最大任务并行度。默认值是`5`。此参数是一个FE动态参数。更多信息和配置方法，请参见[参数配置](../../administration/FE_configuration.md#loading-and-unloading)。

因此，当待消费的分区数和存活的BE节点数大于其他两个参数时，您可以增加desired_concurrent_number和max_routine_load_task_concurrent_num参数的值，以提升实际加载任务的并行度。

例如，待消费的分区数为7，存活的BE节点数为5，max_routine_load_task_concurrent_num的默认值为5。此时，如果您希望将加载任务的并行度提升到最大限度，您需要将desired_concurrent_number设置为5（默认值为3）。然后，实际的任务并行度计算为min(5,7,5,5)，得出5。

更多参数描述，请参见[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。

**方法2：增加常规加载任务从一个或多个分区消耗的数据量。**

> **注意**
> 这种方法可能会导致数据加载**延迟**。

常规加载任务可以消费的消息数量上限，由参数`max_routine_load_batch_size`（表示加载任务可以消费的最大消息数量）或参数`routine_load_task_consume_second`（表示消息消费的最大持续时间）决定。一旦加载任务消费了足够的数据以满足任一要求，该消费过程即告完成。这两个参数是FE动态参数。更多信息和配置方法，请参见[参数配置](../../administration/FE_configuration.md#loading-and-unloading)。

您可以通过查看**be/log/be.INFO**日志来分析决定加载任务消费数据量上限的是哪个参数。通过提高该参数，您可以增加加载任务消费的数据量。

```Plaintext
I0325 20:27:50.410579 15259 data_consumer_group.cpp:131] consumer group done: 41448fb1a0ca59ad-30e34dabfa7e47a0. consume time(ms)=3261, received rows=179190, received bytes=9855450, eos: 1, left_time: -261, left_bytes: 514432550, blocking get time(us): 3065086, blocking put time(us): 24855
```

通常情况下，日志中的left_bytes字段大于或等于0，表明加载任务在routine_load_task_consume_second内消费的数据量没有超出max_routine_load_batch_size。这意味着一批计划的加载任务能够消费Kafka中的所有数据，不会造成消费延迟。在这种情况下，您可以将routine_load_task_consume_second设置为更大的值，以增加加载任务从一个或多个分区消费的数据量。

如果left_bytes字段小于0，这意味着加载任务在routine_load_task_consume_second内消费的数据量已达到max_routine_load_batch_size。每次Kafka中的数据都会填满一批计划的加载任务。因此，很可能Kafka中还有未被消费的剩余数据，导致消费延迟。在这种情况下，您可以将max_routine_load_batch_size设置为更大的值。

## 如果SHOW ROUTINE LOAD的结果显示加载作业处于PAUSED状态，我该怎么办？

- 检查ReasonOfStateChanged字段，它报告了错误消息“Broker: Offset out of range”。

  **原因分析：**加载作业的消费者偏移量在Kafka分区中不存在。

  **解决方案:** 您可以执行[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)和检查参数`Progress`中的加载作业的最新消费者偏移量。然后，您可以验证对应的消息是否存在于Kafka分区中。如果不存在，可能是因为

  - 创建加载作业时指定的消费者偏移量是一个未来的偏移量。
  - Kafka分区中指定的消费者偏移处的消息在加载作业消费之前已被删除。建议基于加载速度，设置合理的Kafka日志清理策略和参数，例如log.retention.hours和log.retention.bytes。

- 如果检查ReasonOfStateChanged字段，它没有报告错误消息“Broker: Offset out of range”。

  **原因分析:** 加载任务的错误行数超过了阈值 `max_error_number`。

  **解决方案:** 您可以利用`ReasonOfStateChanged`和`ErrorLogUrls`字段中的错误消息来排查并修复问题。

-   如果是由于数据源中的数据格式不正确引起的，您需要检查数据格式并解决问题。成功解决问题后，您可以使用[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)来恢复暂停的加载作业。

-   如果是因为StarRocks无法解析数据源中的数据格式，您需要调整阈值 `max_error_number`。您可以先执行[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)来查看 `max_error_number` 的当前值，然后使用[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)来提高该阈值。修改阈值后，您可以使用[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)来恢复暂停的加载作业。

## 如果SHOW ROUTINE LOAD的结果显示加载作业处于CANCELLED状态，我该怎么办？

**原因分析：**加载作业在加载过程中遇到异常，例如表被删除。

**解决方案:** 在排查和修复问题时，您可以参考错误消息在`ReasonOfStateChanged`和`ErrorLogUrls`字段中的。但是，在修复问题后，您无法恢复已取消的加载作业。

## 常规加载在从Kafka消费并写入StarRocks时能否保证一致性语义？

常规加载保证了精确一次(exactly-once)的语义。

每个加载任务都是一个独立的事务。如果事务执行过程中发生错误，则事务将被中止，FE不会更新加载任务相关分区的消费进度。当FE下次从任务队列中调度加载任务时，这些任务会从分区最后保存的消费位置发起消费请求，从而确保了精确一次的语义。

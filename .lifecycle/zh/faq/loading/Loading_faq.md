---
displayed_sidebar: English
---

# 数据加载

## 1. 如果出现“关闭索引通道失败”或“平板电脑版本过多”的错误该怎么办？

当您运行的加载作业过于频繁，且数据未能及时进行压缩时，就会导致在加载过程中生成的数据版本数量超过了系统允许的最大值（默认为1000）。您可以使用以下方法之一来解决此问题：

- - 增加每个作业中加载的数据量，以此降低加载的频率。

- - 修改每个BE节点的BE配置文件**be.conf**中的配置项，以加快数据压缩的速度：

  ```Plain
  cumulative_compaction_num_threads_per_disk = 4
  base_compaction_num_threads_per_disk = 2
  cumulative_compaction_check_interval_seconds = 2
  ```

  修改上述配置项后，您必须监控内存和I/O，确保它们处于正常状态。

## 2. What do I do if the "Label Already Exists" error occurs?

2. 如果出现“标签已存在”的错误该怎么办？

这个错误是因为您的加载作业与同一个StarRocks数据库中已成功运行或正在运行的另一个加载作业拥有相同的标签。Stream Load作业是通过HTTP提交的。通常，所有编程语言的HTTP客户端都内置了请求重试逻辑。当StarRocks集群从HTTP客户端接收到加载作业请求时，会立即开始处理该请求，但不会及时向HTTP客户端返回作业结果。因此，HTTP客户端会再次发送相同的加载作业请求。然而，StarRocks集群已经在处理第一个请求，所以会对第二个请求返回“标签已存在”的错误。

为了检查使用不同加载方法提交的加载作业是否具有相同的标签，并且没有被重复提交，请执行以下操作：

- - 查看FE日志，检查失败的加载作业的标签是否被记录了两次。如果是，说明客户端提交了两次相同的加载作业请求。

    > **注意**：StarRocks集群不会根据加载方式区分加载作业的标签。因此，使用不同加载方法提交的加载作业可能会有相同的标签。
    > StarRocks集群不区分基于加载方法的标签。因此，使用不同加载方法提交的加载作业可能具有相同的标签。

- Run SHOW LOAD WHERE LABEL = "xxx" to check for load jobs that have the same label and are in the **FINISHED** state.

    > **在提交加载作业之前，我们建议您预估加载数据所需的时间，并相应地调整客户端请求的超时时间。这样可以防止客户端多次提交相同的加载作业请求。**
    > `xxx` is the label that you want to check.

3. 如果出现“ETL_QUALITY_UNSATISFIED; msg:quality not good enough to cancel”的错误，该怎么办？

## 执行SHOW LOAD，并使用返回结果中的错误URL来查看错误的详细信息。

执行[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)，并使用返回的执行结果中的错误URL查看错误详情。

- “将csv字符串转换为INT失败。”

- 无法将源列中的字符串转换为目标列所匹配的数据类型。例如，无法将abc转换为数值。

  - “输入的长度比模式定义的长度长。”

- 源列中的值长度超出了目标列在表创建时定义的最大长度。例如，CHAR数据类型的源列值超过了目标列的最大长度，或INT数据类型的源列值超过了4字节。

  - “实际列数小于表架构定义的列数。”

- 根据指定的列分隔符解析源行后，得到的列数少于目标表的列数。可能的原因是加载命令或语句中指定的列分隔符与实际使用的列分隔符不同。

  - “实际列数大于表架构定义的列数。”

- 根据指定的列分隔符解析源行后，得到的列数多于目标表的列数。可能的原因是加载命令或语句中指定的列分隔符与实际使用的列分隔符不同。

  - “小数部分的长度超过了模式定义的比例。”

- DECIMAL类型源列中的值的小数部分超出了指定的长度。

  - “整数部分的长度超过了模式定义的精度。”

- DECIMAL类型源列中的值的整数部分超出了指定的长度。

  - “对于此键没有相应的分区。”

- 源行的分区列的值不在分区范围内。

  The value in the partition column for a source row is not within the partition range.

## 4. 如果RPC超时该怎么办？

检查每个BE节点的BE配置文件**be.conf**中的`write_buffer_size`配置项。此配置项用于控制BE上每个内存块的最大尺寸，默认最大尺寸为100 MB。如果设置的尺寸过大，可能会导致远程过程调用（RPC）超时。要解决此问题，调整BE配置文件中的`write_buffer_size`和`tablet_writer_rpc_timeout_sec`配置项。更多信息，请参见[BE配置](../../loading/Loading_intro.md#be-configurations)。

## 5. What do I do if the "Value count does not match column count" error occurs?

5. 如果出现“值的数量与列的数量不匹配”的错误该怎么办？

```Java
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:00Z,cpu0,80.99
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:10Z,cpu1,75.23
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:20Z,cpu2,59.44
```

我的加载作业失败后，我使用作业结果中返回的错误URL来检索错误详细信息，并发现了“值的数量与列的数量不匹配”的错误。这表明源数据文件中的列数与目标StarRocks表中的列数不一致：

这个问题的原因是加载命令或语句中指定的列分隔符与源数据文件中实际使用的列分隔符不同。在上述示例中，CSV格式的数据文件包含三列，列与列之间用逗号（,）分隔。但是，在加载命令或语句中指定的列分隔符是制表符（\t）。因此，源数据文件中的三列被错误地解析成了一列。

请在加载命令或语句中指定逗号（,）作为列分隔符，然后重新提交加载作业。

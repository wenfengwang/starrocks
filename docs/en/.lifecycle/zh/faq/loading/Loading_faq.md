---
displayed_sidebar: English
---

# 数据加载

## 1. 如果出现“关闭索引通道失败”或“tablet版本过多”错误怎么办？

您的加载作业运行过于频繁，数据未能及时进行压缩。这导致在加载过程中生成的数据版本数量超过了允许的最大数据版本数（默认为1000）。使用以下方法之一解决此问题：

- 增加每个作业中加载的数据量，以此减少加载频率。

- 修改每个BE的BE配置文件 **be.conf** 中的配置项，如下所示，以加快数据压缩：

  ```Plain
  cumulative_compaction_num_threads_per_disk = 4
  base_compaction_num_threads_per_disk = 2
  cumulative_compaction_check_interval_seconds = 2
  ```

  修改上述配置项后，您必须监控内存和I/O，确保它们处于正常状态。

## 2. 如果出现“Label Already Exists”错误怎么办？

该错误是因为加载作业的标签与同一个StarRocks数据库中已成功运行或正在运行的另一个加载作业的标签相同。

Stream Load作业是通过HTTP提交的。通常，所有编程语言的HTTP客户端都内置了请求重试逻辑。当StarRocks集群从HTTP客户端接收到加载作业请求时，它会立即开始处理该请求，但不会及时向HTTP客户端返回作业结果。因此，HTTP客户端会再次发送相同的加载作业请求。然而，StarRocks集群已经在处理第一个请求，因此对第二个请求返回`Label Already Exists`错误。

为了检查使用不同加载方法提交的加载作业是否有相同的标签且没有重复提交，请执行以下操作：

- 查看FE日志，检查失败的加载作业的标签是否被记录了两次。如果标签记录了两次，说明客户端提交了两次加载作业请求。

    > **注意**
    > StarRocks集群不会根据加载方法区分加载作业的标签。因此，使用不同加载方法提交的加载作业可能会有相同的标签。

- 运行 SHOW LOAD WHERE LABEL = "xxx" 来检查是否有具有相同标签且处于 **FINISHED** 状态的加载作业。

    > **注意**
    > `xxx` 是您想要检查的标签。

在提交加载作业之前，我们建议您估算加载数据所需的大致时间，然后相应地调整客户端请求的超时时间。这样可以防止客户端重复提交加载作业请求。

## 3. 如果出现“ETL_QUALITY_UNSATISFIED; msg:quality not good enough to cancel”错误，该怎么办？

执行 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)，并使用返回结果中的错误URL查看错误详情。

常见的数据质量错误包括：

- “转换csv字符串到INT失败。”

  来自源列的字符串未能转换成匹配目标列的数据类型。例如，`abc` 无法转换成数值。

- “输入长度超过模式定义。”

  来自源列的值长度超出了匹配目标列支持的长度。例如，CHAR数据类型的源列值超过了表创建时指定的目标列的最大长度，或者INT数据类型的源列值超过了4字节。

- “实际列数少于模式定义的列数。”

  根据指定的列分隔符解析源行后，得到的列数少于目标表的列数。可能的原因是加载命令或语句中指定的列分隔符与实际使用的列分隔符不同。

- “实际列数多于模式定义的列数。”

  根据指定的列分隔符解析源行后，得到的列数多于目标表的列数。可能的原因是加载命令或语句中指定的列分隔符与实际使用的列分隔符不同。

- “小数部分长度超过模式定义的比例。”

  来自DECIMAL类型源列的值的小数部分超出了指定的长度。

- “整数部分长度超过模式定义的精度。”

  来自DECIMAL类型源列的值的整数部分超出了指定的长度。

- “此键没有对应分区。”

  源行的分区列值不在分区范围内。

## 4. 如果RPC超时该怎么办？

检查每个BE的BE配置文件 **be.conf** 中的`write_buffer_size`配置项。该配置项用于控制BE上每个内存块的最大尺寸，默认最大尺寸为100MB。如果最大尺寸设置得过大，可能会导致远程过程调用（RPC）超时。解决此问题，需要调整BE配置文件中的`write_buffer_size`和`tablet_writer_rpc_timeout_sec`配置项。更多信息，请参见 [BE配置](../../loading/Loading_intro.md#be-configurations)。

## 5. 如果出现“值数量与列数量不匹配”错误该怎么办？

我的加载作业失败后，我使用作业结果中返回的错误URL来检索错误详情，发现了“值数量与列数量不匹配”的错误，这表明源数据文件中的列数与目标StarRocks表中的列数不一致：

```Java
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:00Z,cpu0,80.99
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:10Z,cpu1,75.23
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:20Z,cpu2,59.44
```

问题原因如下：

加载命令或语句中指定的列分隔符与源数据文件中实际使用的列分隔符不一致。在上述示例中，CSV格式的数据文件包含三列，这些列之间用逗号（`,`）分隔。但是，在加载命令或语句中指定的列分隔符是制表符（`\t`）。因此，源数据文件中的三列被错误地解析为一列。

在加载命令或语句中指定逗号（`,`）作为列分隔符，然后重新提交加载作业。
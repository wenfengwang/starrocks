
- 使用 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) + [`FILES()`](../../sql-reference/sql-functions/table-functions/files.md) 进行同步加载
- 使用 [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 进行异步加载
- 使用 [Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md) 进行连续异步加载

这些选项各有其优点，详细内容将在以下章节中介绍。

在大多数情况下，我们建议您使用 INSERT+`FILES()` 方法，这种方法更易于使用。

然而，INSERT+`FILES()` 方法目前仅支持 Parquet 和 ORC 文件格式。因此，如果您需要加载其他文件格式的数据，比如 CSV，或者在数据加载过程中执行类似 DELETE 的数据更改操作，您可以使用 Broker Load。

如果您需要加载大量数据文件，总数据量相当大（例如超过 100 GB，甚至 1 TB），我们建议您使用 Pipe 方法。Pipe 可以根据文件数量或大小拆分文件，将加载作业分解为较小的顺序任务。这种方法可以确保一个文件中的错误不会影响整个加载作业，并最大程度地减少由于数据错误而需要重试的次数。

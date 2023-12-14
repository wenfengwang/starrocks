
- 使用[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../../sql-reference/sql-functions/table-functions/files.md)进行同步加载
- 使用[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)进行异步加载
- 使用[Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)进行持续异步加载

每种选项都有其自己的优势，这些将在下面的章节中详细介绍。

在大多数情况下，我们建议您使用INSERT+`FILES()`的方法，这种方法更容易使用。

然而，目前INSERT+`FILES()`仅支持Parquet和ORC文件格式。因此，如果您需要加载其他文件格式（如CSV）的数据，或者需要在数据加载过程中进行[数据更改，如删除操作](../../loading/Load_to_Primary_Key_tables.md)，您可以转而使用Broker Load。

如果您需要加载大量数据文件，其总数据量较大（例如，超过100GB甚至1TB），我们建议您使用Pipe方法。Pipe可以根据文件的数量或大小将文件拆分，将加载作业分解为较小的顺序任务。这种方法确保了一个文件中的错误不会影响整个加载作业，并减少了由于数据错误而需要重试的情况。
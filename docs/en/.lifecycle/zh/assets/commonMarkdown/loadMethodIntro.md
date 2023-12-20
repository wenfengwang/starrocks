
- 使用 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../../sql-reference/sql-functions/table-functions/files.md) 进行同步加载
- 使用 [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 进行异步加载
- 使用 [Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md) 进行持续异步加载

每种选项都有其独特的优势，在以下各节中将详细介绍。

在大多数情况下，我们推荐您使用 INSERT+`FILES()` 方法，它的使用更为简便。

然而，INSERT+`FILES()` 方法目前只支持 Parquet 和 ORC 文件格式。因此，如果您需要加载 CSV 等其他文件格式的数据，或者需要在数据加载过程中[执行 DELETE 等数据变更操作](../../loading/Load_to_Primary_Key_tables.md)，您可以使用 Broker Load。

如果您需要加载大量的数据文件，且总数据量相当大（例如，超过 100 GB 或甚至 1 TB），我们建议您使用 Pipe 方法。Pipe 能够根据文件数量或大小分割文件，把加载任务分解成更小的、顺序执行的任务。这种方式确保单个文件的错误不会影响整个加载作业，并且能够最小化因数据错误需要重试的情况。
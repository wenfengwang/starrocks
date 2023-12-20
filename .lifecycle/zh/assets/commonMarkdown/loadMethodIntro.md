
- Synchronous loading using [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../../sql-reference/sql-functions/table-functions/files.md)进行同步加载
- 使用[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)实现异步加载
- 利用[Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)进行持续的异步加载

以上每种选项都各有优势，在接下来的章节中将详细阐述。

在大多数情况下，我们推荐您采用INSERT+FILES()方法，因为它更加简便易用。

然而，目前`INSERT+FILES()`方法只支持Parquet和ORC文件格式。因此，如果您需要加载数据的其他文件格式，比如CSV，或者[在数据加载过程中执行DELETE等数据变更操作](../../loading/Load_to_Primary_Key_tables.md)，您可以考虑使用Broker Load。

如果您需要加载的数据文件数量众多，并且总体数据量很大（比如超过100GB或甚至1TB），我们推荐您使用Pipe方法。Pipe能够基于文件数量或大小将文件分割，把加载任务分解为一系列更小的顺序任务。这种方式确保了单个文件的错误不会影响整个加载作业，并且能够最小化因数据错误需要重试的情况。

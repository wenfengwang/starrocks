
- [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../../sql-reference/sql-functions/table-functions/files.md)を使用した同期ロード
- [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ロード
- [Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を使用した連続非同期ロード

これらのオプションにはそれぞれ独自の利点があり、次のセクションで詳しく説明します。

ほとんどの場合、INSERT+`FILES()`メソッドを使用することをお勧めします。これははるかに使いやすいです。

ただし、INSERT+`FILES()`メソッドは現在、ParquetおよびORCファイル形式のみをサポートしています。したがって、CSVなどの他のファイル形式のデータを読み込む必要がある場合や、[データの読み込み中にDELETEなどのデータ変更を実行する必要がある場合](../../loading/Load_to_Primary_Key_tables.md)は、Broker Loadを利用できます。

合計で大量のデータボリューム（たとえば、100 GB以上、あるいは1 TB以上）を持つ多数のデータファイルを読み込む必要がある場合は、Pipeメソッドを使用することをお勧めします。Pipeはファイルの数またはサイズに基づいてファイルを分割し、ロードジョブをより小さく、連続したタスクに分割することができます。このアプローチにより、1つのファイルのエラーがロードジョブ全体に影響を与えることがなくなり、データエラーによる再試行の必要性が最小限に抑えられます。

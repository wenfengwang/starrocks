- [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../../sql-reference/sql-functions/table-functions/files.md)を使用した同期的なローディング
- [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ローディング
- [Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を使用した連続的な非同期ローディング

それぞれのオプションにはそれぞれの利点があり、以下のセクションで詳細に説明されています。

ほとんどの場合、INSERT+`FILES()`メソッドの使用を推奨します。これははるかに使いやすいからです。

ただし、INSERT+`FILES()`メソッドでは現在、ParquetおよびORCファイルフォーマットのみがサポートされています。そのため、CSVなどの他のファイル形式のデータをロードする必要がある場合や、[データローディング中の削除などのデータ変更を実行する必要がある場合](../../loading/Load_to_Primary_Key_tables.md)は、Broker Loadを利用できます。

合計データ量が大きい（たとえば、100 GB以上、1 TBである場合）大量のデータファイルをロードする必要がある場合、Pipeメソッドの使用を推奨します。Pipeはファイルを数やサイズに基づいて分割し、ロードジョブをより小さな連続的なタスクに分割できます。このアプローチにより、1つのファイルのエラーがすべてのロードジョブに影響を与えることはありませんし、データエラーによるリトライの必要性を最小限に抑えることができます。
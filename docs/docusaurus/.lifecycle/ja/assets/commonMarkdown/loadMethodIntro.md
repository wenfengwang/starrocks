- [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../../sql-reference/sql-functions/table-functions/files.md)を使用した同期ローディング
- [ブローカーローディング](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ローディング
- [Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を使用した連続非同期ローディング

それぞれのオプションには、次のセクションで詳細に説明されている利点があります。

ほとんどの場合、より簡単に使用できるINSERT+`FILES()`メソッドをお勧めします。

ただし、INSERT+`FILES()`メソッドは現在、ParquetおよびORCファイル形式のみをサポートしています。そのため、CSVなどの他のファイル形式のデータをロードする必要がある場合や、[データローディング中の削除などのデータ変更を実行](../../loading/Load_to_Primary_Key_tables.md)する必要がある場合は、ブローカーローディングに頼ることができます。

合計データ量が多い（たとえば、100 GB以上、または1 TBであるなど）データファイルを大量にロードする必要がある場合は、Pipeメソッドを使用することをお勧めします。Pipeはファイルをその数やサイズに基づいて分割し、ロードジョブをより小さな連続的なタスクに分割できます。このアプローチにより、1つのファイルでのエラーがロードジョブ全体に影響を与えることはありませんし、データエラーによる再試行の必要性を最小限に抑えることができます。
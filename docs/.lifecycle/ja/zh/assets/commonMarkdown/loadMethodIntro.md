
- 同期インポートには [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../../sql-reference/sql-functions/table-functions/files.md) を使用します。
- 非同期インポートには [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を使用します。
- 継続的な非同期インポートには [Pipe](../../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md) を使用します。

これら三つのインポート方法にはそれぞれ利点があり、以下の各セクションで詳しく説明します。

一般的には、INSERT+`FILES()` を使用することをお勧めします。これはより便利で使いやすいです。

しかし、INSERT+`FILES()` は現在 Parquet と ORC ファイルフォーマットのみをサポートしています。そのため、他のフォーマット（例えば CSV）のデータをインポートする必要がある場合や、インポートプロセス中に [DELETE などのデータ変更操作を実行する必要がある場合](../../loading/Load_to_Primary_Key_tables.md)は、Broker Load を使用することができます。

非常に大きなデータ（例えば 100 GB を超える、特に 1 TB 以上のデータ量）をインポートする必要がある場合は、Pipe を使用することをお勧めします。Pipe はファイルの数やサイズに応じて、ディレクトリ内のファイルを自動的に分割し、大きなインポートジョブを複数の小さな連続したインポートタスクに分割します。これにより、エラーが発生した際のリトライのコストを低減します。また、継続的なデータインポートを行う際にも Pipe を使用することを推奨します。これはリモートストレージディレクトリのファイル変更を監視し、変更されたファイルデータを継続的にインポートすることができます。

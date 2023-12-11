---
displayed_sidebar: "Japanese"
---

# その他のFAQ

このトピックでは、一般的な質問に対する回答を提供します。

## VARCHAR（32）とSTRINGは同じストレージスペースを占有しますか？

両方が可変長のデータ型です。同じ長さのデータを保存する場合、VARCHAR（32）とSTRINGは同じストレージスペースを占有します。

## VARCHAR（32）とSTRINGはデータクエリで同じパフォーマンスを発揮しますか？

はい。

## OracleからインポートされたTXTファイルの文字セットをUTF-8に設定しても文字化けが解消されない理由は何ですか？

この問題を解決するには、次の手順を実行してください。

1. 例えば、**original** という名前のファイルがあり、そのテキストが文字化けしています。このファイルの文字セットはISO-8859-1です。次のコードを実行してファイルの文字セットを取得してください。

    ```plaintext
    file --mime-encoding origin.txt
    origin.txt: iso-8859-1
    ```

2. `iconv` コマンドを実行して、このファイルの文字セットをUTF-8に変換してください。

    ```plaintext
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. 変換後、このファイルのテキストが文字化けしたままの場合は、ファイルの文字セットをGBKに再度変更してUTF-8に再変換してください。

    ```plaintext
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## MySQLでのSTRINGの長さは、StarRocksでのSTRINGの長さと同じですか？

VARCHAR(n)の場合、StarRocksは「n」をバイト単位で定義し、MySQLは「n」を文字単位で定義します。UTF-8によると、1つの中国語文字は3バイトに相当します。StarRocksとMySQLが同じ数値を「n」として定義する場合、MySQLはStarRocksの3倍の文字を保存します。

## テーブルのパーティションフィールドのデータ型には、FLOAT、DOUBLE、DECIMALを使用できますか？

いいえ、DATE、DATETIME、およびINTのみがサポートされています。

## テーブル内のデータが占有するストレージスペースを確認する方法は？

SHOW DATA ステートメントを実行して対応するストレージスペースを表示します。データのボリューム、コピー数、および行数も表示できます。

**注意**: データ統計には時間の遅れがあります。

## StarRocksデータベースのクォータを増やすにはどのようにリクエストを送信しますか？

クォータを増やすには、次のコードを実行してください。

```plaintext
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocksは、UPSERTステートメントを実行してテーブル内の特定のフィールドを更新することをサポートしていますか？

StarRocks 2.2以降では、プライマリキー テーブルを使用してテーブル内の特定のフィールドを更新することができます。StarRocks 1.9以降では、プライマリキーテーブルを使用してテーブル内のすべてのフィールドを更新することができます。詳細については、StarRocks 2.2の [プライマリキーテーブル](../table_design/table_types/primary_key_table.md) を参照してください。

## 2つのテーブルまたはパーティション間でデータを交換する方法は？

SWAP WITH ステートメントを実行して、2つのテーブルまたは2つのパーティション間でデータを交換します。SWAP WITH ステートメントは INSERT OVERWRITE ステートメントよりも安全です。データを交換する前にデータを確認し、交換後のデータが交換前のデータと一致するかどうかを確認してからデータを交換してください。

- 2つのテーブルを交換する：例えば、table 1という名前のテーブルがある場合、table 1を別のテーブルで置き換えたい場合は、次の手順を実行してください。

    1. table 2という名前の新しいテーブルを作成してください。

        ```SQL
        create table2 like table1;
        ```

   2. Stream Load、Broker Load、またはInsert Intoを使用して、table 1からtable 2にデータをロードしてください。

    3. table 1をtable 2で置き換えてください。

        ```SQL
        ALTER TABLE table1 SWAP WITH table2;
        ```

        これにより、データが正確にtable 1にロードされます。

- 2つのパーティションを交換する：例えば、table 1という名前のテーブルがある場合、table 1内のパーティションデータを置き換えたい場合は、次の手順を実行してください。

    1. 一時的なパーティションを作成してください。

        ```SQL
        ALTER TABLE table1

        ADD TEMPORARY PARTITION tp1

        VALUES LESS THAN("2020-02-01");
        ```

    2. table 1内のパーティションデータを一時的なパーティションにロードしてください。

    3. table 1のパーティションを一時的なパーティションで置き換えてください。

        ```SQL
        ALTER TABLE table1

        REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
        ```

## フロントエンド（FE）を再起動すると、"error to open replicated environment, will exit" というエラーが発生します。

このエラーはBDBJEのバグに起因します。この問題を解決するには、BDBJEのバージョンを1.17以降に更新してください。

## 新しいApache Hiveテーブルからデータをクエリする際に "Broker list path exception" というエラーが発生します

### 問題の説明

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解決策

StarRocksテクニカルサポートに連絡し、namenodeのアドレスとポートが正しいかどうか、およびnamenodeのアドレスとポートにアクセス権があるかどうかを確認してください。

## 新しいApache Hiveテーブルからデータをクエリする際に "get hive partition metadata failed" というエラーが発生します

### 問題の説明

```plaintext
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解決策

ネットワークが接続されていることを確認し、各バックエンド（BE）に **host** ファイルをアップロードしてください。

## Apache Hiveの外部テーブルへのアクセス時に "do_open failed. reason = Invalid ORC postscript length" というエラーが発生します

### 問題の説明

Apache HiveのメタデータはFEにキャッシュされます。しかし、StarRocksはメタデータを更新するには2時間かかります。StarRocksが更新を完了する前に、Apache Hiveテーブルに新しいデータを挿入したりデータを更新したりすると、BEがスキャンしたHDFSのデータと、FEが取得したデータが異なるため、このエラーが発生します。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解決策

この問題を解決するには、次のいずれかの操作を実行してください。

- 現在のバージョンをStarRocks 2.2またはそれ以降にアップグレードします。
- Apache Hiveテーブルを手動で更新します。詳細については、[メタデータキャッシュ戦略](../data_source/External_table.md)を参照してください。

## MySQLの外部テーブルに接続する際に "caching_sha2_password cannot be loaded" というエラーが発生します

### 問題の説明

MySQL 8.0のデフォルト認証プラグインはcaching_sha2_passwordです。MySQL 5.7のデフォルト認証プラグインはmysql_native_passwordです。間違った認証プラグインを使用したため、このエラーが発生しています。

### 解決策

この問題を解決するには、次のいずれかの操作を実行してください。

- StarRocksに接続してください。

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

- `my.cnf`ファイルを編集してください。

```plaintext
vim my.cnf

[mysqld]

default_authentication_plugin=mysql_native_password
```

## テーブルを削除した後、ディスクスペースをすぐに解放するにはどのようにすればよいですか？

テーブルを削除するには、DROP TABLE ステートメントを実行しても、StarRocksは割り当てられたディスクスペースを解放するまで時間がかかります。割り当てられたディスクスペースをすぐに解放するには、DROP TABLE FORCE ステートメントを実行してテーブルを削除してください。DROP TABLE FORCE ステートメントを実行すると、StarRocksは未完了のイベントがあるかどうかを確認せずにテーブルを直接削除します。DROP TABLE FORCE ステートメントを慎重に実行することをお勧めします。なぜなら、テーブルが削除されると、それを復元することはできないからです。

## 現在のStarRocksのバージョンを確認するにはどうすればよいですか？

`select current_version();` コマンドを実行するか、CLIコマンド `./bin/show_fe_version.sh` を実行して現在のバージョンを確認してください。

## FEのメモリサイズを設定するにはどうすればよいですか？

FEが使用するメモリにメタデータが保存されます。以下の表に示すように、FEあたりのタブレットの数に応じてFEのメモリサイズを設定できます。例えば、タブレットの数が100万未満の場合、FEに最低16 GBのメモリを割り当てる必要があります。 **fe.conf** ファイルの **JAVA_OPTS** 構成項目内のパラメータ **-Xms** と **-Xmx** の値を構成してください。注意:すべてのFEでの構成は同じでなければなりません。FEのいずれかがリーダーとして選出される可能性があるためです。

| タブレットの数 | 各FEのメモリサイズ |
| -------------- | ----------- |
| 100万未満       | 16 GB        |
| 100万～200万   | 32 GB        |
| 200万～500万   | 64 GB        |
| 500万～1000万   | 128 GB       |
## StarRocksのクエリ時間はどのように計算されますか？

StarRocksは複数のスレッドを使用してデータをクエリすることをサポートしています。クエリ時間とは、複数のスレッドがデータをクエリするのにかかる時間を指します。

## StarRocksはデータをローカルにエクスポートする際にパスの設定をサポートしていますか？

いいえ。

## StarRocksの同時実行上限は何ですか？

実際のビジネスシナリオまたはシミュレートされたビジネスシナリオに基づいて同時実行の制限をテストできます。一部のユーザーのフィードバックによると、最大で20,000 QPSまたは30,000 QPSまで達成できます。

## StarRocksの初回のSSBテストパフォーマンスが2回目よりも遅いのはなぜですか？

最初のクエリでディスクの読み取り速度はディスクのパフォーマンスに関連しています。最初のクエリの後、次のクエリのためにページキャッシュが生成されるため、クエリは以前よりも速くなります。

## クラスターに少なくとも何台のBEを構成する必要がありますか？

StarRocksは単一ノードの展開をサポートしているため、少なくとも1つのBEを構成する必要があります。BEはAVX2で実行する必要があるため、8コアおよび16GB以上の構成のマシンにBEを展開することをお勧めします。

## Apache Supersetを使用してStarRocksでデータを可視化する際に、データ権限をどのように設定しますか？

新しいユーザーアカウントを作成し、その後、テーブルクエリに対するユーザーへの権限を付与することでデータ権限を設定できます。

## `enable_profile`を`true`に設定した後、なぜプロファイルの表示が失敗するのですか？

レポートはリーダーFEにのみ提出されます。

## StarRocksのテーブル内のフィールド注釈をチェックする方法は？

`show create table xxx`コマンドを実行します。

## テーブルを作成する際、NOW()関数のデフォルト値を指定する方法は？

StarRocks 2.1以降のバージョンのみが関数のデフォルト値を指定することをサポートしています。StarRocks 2.1より前のバージョンでは、関数の定数しか指定できません。

## BEノードのストレージスペースを解放するにはどうすればいいですか？

`rm -rf`コマンドを使用して`trash`ディレクトリを削除します。スナップショットからデータを復元済みの場合は、`snapshot`ディレクトリを削除できます。

## BEノードに追加のディスクを追加できますか？

はい。BE構成項目`storage_root_path`で指定されたディレクトリにディスクを追加できます。
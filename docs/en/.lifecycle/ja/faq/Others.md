---
displayed_sidebar: "Japanese"
---

# その他のFAQ

このトピックでは、いくつかの一般的な質問に対する回答を提供します。

## VARCHAR (32) と STRING は同じストレージスペースを占有しますか？

どちらも可変長のデータ型です。同じ長さのデータを格納する場合、VARCHAR (32) と STRING は同じストレージスペースを占有します。

## VARCHAR (32) と STRING はデータクエリにおいて同じパフォーマンスを発揮しますか？

はい。

## Oracle からインポートした TXT ファイルが UTF-8 の文字セットに設定されているにもかかわらず、まだ文字化けして表示されます。なぜですか？

この問題を解決するには、次の手順を実行します。

1. 例えば、**original** という名前のファイルがあり、そのテキストが文字化けしています。このファイルの文字セットは ISO-8859-1 です。次のコードを実行して、ファイルの文字セットを取得します。

    ```plaintext
    file --mime-encoding origin.txt
    origin.txt: iso-8859-1
    ```

2. `iconv` コマンドを実行して、このファイルの文字セットを UTF-8 に変換します。

    ```plaintext
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. 変換後、このファイルのテキストはまだ文字化けして表示されます。その後、このファイルの文字セットを GBK に変更し、再度文字セットを UTF-8 に変換します。

    ```plaintext
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## MySQL によって定義される STRING の長さは、StarRocks によって定義される STRING の長さと同じですか？

VARCHAR(n) の場合、StarRocks は「n」をバイト単位で定義し、MySQL は「n」を文字単位で定義します。UTF-8 によると、1つの中国語の文字は3バイトに相当します。StarRocks と MySQL が「n」を同じ数値で定義する場合、MySQL は StarRocks の3倍の文字を保存します。

## テーブルのパーティションフィールドのデータ型は FLOAT、DOUBLE、または DECIMAL にすることはできますか？

いいえ、DATE、DATETIME、および INT のみがサポートされています。

## テーブル内のデータが占有するストレージスペースを確認するにはどうすればよいですか？

SHOW DATA ステートメントを実行して、対応するストレージスペースを確認します。データのボリューム、コピーの数、および行数も表示できます。

**注意**: データの統計にはタイムディレイがあります。

## StarRocks データベースのクォータを増やすにはどうすればよいですか？

クォータを増やすには、次のコードを実行します。

```plaintext
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocks は UPSERT ステートメントを実行してテーブルの特定のフィールドを更新することをサポートしていますか？

StarRocks 2.2 以降では、プライマリキーテーブルを使用してテーブルの特定のフィールドを更新することがサポートされています。StarRocks 1.9 以降では、プライマリキーテーブルを使用してテーブルのすべてのフィールドを更新することがサポートされています。詳細については、StarRocks 2.2 の [プライマリキーテーブル](../table_design/table_types/primary_key_table.md) を参照してください。

## 2つのテーブルまたは2つのパーティション間でデータを交換するにはどうすればよいですか？

SWAP WITH ステートメントを実行して、2つのテーブルまたは2つのパーティション間でデータを交換します。SWAP WITH ステートメントは INSERT OVERWRITE ステートメントよりも安全です。データを交換する前に、まずデータを確認し、データの交換後にデータがデータの交換前と一致しているかどうかを確認します。

- 2つのテーブルを交換する場合: 例えば、table1 という名前のテーブルがある場合、table1 を別のテーブルで置き換えたい場合、次の手順を実行します。

    1. table2 という名前の新しいテーブルを作成します。

        ```SQL
        create table2 like table1;
        ```

    2. Stream Load、Broker Load、または Insert Into を使用して、table1 から table2 にデータをロードします。

    3. table1 を table2 で置き換えます。

        ```SQL
        ALTER TABLE table1 SWAP WITH table2;
        ```

        これにより、データが正確に table1 にロードされます。

- 2つのパーティションを交換する場合: 例えば、table1 という名前のテーブルがある場合、table1 のパーティションデータを置き換えたい場合、次の手順を実行します。

    1. 一時パーティションを作成します。

        ```SQL
        ALTER TABLE table1

        ADD TEMPORARY PARTITION tp1

        VALUES LESS THAN("2020-02-01");
        ```

    2. パーティションデータを table1 から一時パーティションにロードします。

    3. table1 のパーティションを一時パーティションで置き換えます。

        ```SQL
        ALTER TABLE table1

        REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
        ```

## フロントエンド (FE) を再起動すると、"error to open replicated environment, will exit" というエラーが発生します

このエラーは、BDBJE のバグによるものです。この問題を解決するには、BDBJE のバージョンを 1.17 以降に更新してください。

## 新しい Apache Hive テーブルからデータをクエリすると、"Broker list path exception" というエラーが発生します

### 問題の説明

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解決策

StarRocks のテクニカルサポートに連絡し、ネームノードのアドレスとポートが正しいか、およびネームノードのアドレスとポートにアクセスする権限があるかどうかを確認してください。

## 新しい Apache Hive テーブルからデータをクエリすると、"get hive partition metadata failed" というエラーが発生します

### 問題の説明

```plaintext
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解決策

ネットワークが接続されていることを確認し、**host** ファイルを StarRocks クラスタの各バックエンド (BE) にアップロードしてください。

## Apache Hive の外部テーブルにアクセスする際に "do_open failed. reason = Invalid ORC postscript length" というエラーが発生します

### 問題の説明

Apache Hive のメタデータは FE にキャッシュされます。しかし、StarRocks はメタデータを更新するために2時間のタイムラグがあります。StarRocks が更新を完了する前に、Apache Hive テーブルに新しいデータを挿入したり、データを更新したりすると、BE によってスキャンされる HDFS のデータと FE によって取得されるデータが異なるため、このエラーが発生します。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解決策

この問題を解決するには、次のいずれかの操作を実行します。

- 現在のバージョンを StarRocks 2.2 以降にアップグレードします。
- Apache Hive テーブルを手動でリフレッシュします。詳細については、[メタデータのキャッシュ戦略](../data_source/External_table.md) を参照してください。

## MySQL の外部テーブルに接続する際に "caching_sha2_password cannot be loaded" というエラーが発生します

### 問題の説明

MySQL 8.0 のデフォルトの認証プラグインは caching_sha2_password です。MySQL 5.7 のデフォルトの認証プラグインは mysql_native_password です。このエラーは、間違った認証プラグインを使用しているために発生します。

### 解決策

この問題を解決するには、次のいずれかの操作を実行します。

- StarRocks に接続します。

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

- `my.cnf` ファイルを変更します。

```plaintext
vim my.cnf

[mysqld]

default_authentication_plugin=mysql_native_password
```

## テーブルを削除した後、すぐにディスクスペースを解放するにはどうすればよいですか？

テーブルを削除するために DROP TABLE ステートメントを実行すると、StarRocks は割り当てられたディスクスペースを解放するまで時間がかかります。割り当てられたディスクスペースをすぐに解放するには、DROP TABLE FORCE ステートメントを実行してテーブルを削除します。DROP TABLE FORCE ステートメントを実行すると、StarRocks はテーブルを確認せずに直接削除します。DROP TABLE FORCE ステートメントを実行する際は注意してください。テーブルが削除されると、元に戻すことはできません。

## StarRocks の現在のバージョンを表示するにはどうすればよいですか？

`select current_version();` コマンドまたは CLI コマンド `./bin/show_fe_version.sh` を実行して、現在のバージョンを表示します。

## FE のメモリサイズを設定するにはどうすればよいですか？

メタデータは FE が使用するメモリに格納されます。FE のメモリサイズは、以下の表に示すように、タブレットの数に応じて設定できます。例えば、タブレットの数が100万以下の場合、FE には最低でも16 GB のメモリを割り当てる必要があります。**fe.conf** ファイルの **JAVA_OPTS** 設定項目のパラメータ `-Xms` と `-Xmx` の値を設定し、パラメータ `-Xms` と `-Xmx` の値を一致させる必要があります。なお、設定はすべての FE で同じである必要があります。なぜなら、いずれの FE でもリーダーとして選出される可能性があるためです。

| タブレットの数    | 各 FE のメモリサイズ |
| -------------- | ----------- |
| 100万以下      | 16 GB        |
| 100万 ～ 200万 | 32 GB        |
| 200万 ～ 500万 | 64 GB        |
| 500万 ～ 1000万   | 128 GB       |

## StarRocks はクエリ時間をどのように計算しますか？

StarRocks は、複数のスレッドを使用してデータをクエリすることができます。クエリ時間とは、複数のスレッドがデータをクエリするために使用する時間のことを指します。

## StarRocks はデータをローカルにエクスポートする際にパスを設定することはできますか？

いいえ。

## StarRocks の同時実行上限は何ですか？

実際のビジネスシナリオまたはシミュレートされたビジネスシナリオに基づいて、同時実行の制限をテストすることができます。一部のユーザーのフィードバックによると、最大で20,000 QPS または 30,000 QPS を達成できると報告されています。

## StarRocks の初回の SSB テストのパフォーマンスが2回目のテストよりも遅いのはなぜですか？

最初のクエリのディスク読み取り速度はディスクのパフォーマンスに関連しています。最初のクエリの後、次のクエリのためにページキャッシュが生成されるため、クエリは以前よりも高速です。

## クラスタには少なくともいくつの BE を設定する必要がありますか？

StarRocks はシングルノードデプロイメントをサポートしているため、少なくとも1つの BE を設定する必要があります。BE は AVX2 で実行する必要があるため、8コアと16GB 以上の構成を持つマシンに BE をデプロイすることをおすすめします。

## Apache Superset を使用して StarRocks のデータを可視化する際にデータの権限を設定する方法はありますか？

新しいユーザーアカウントを作成し、テーブルクエリの権限をユーザーに付与することで、データの権限を設定できます。

## `enable_profile` を `true` に設定した後、プロファイルが表示されなくなるのはなぜですか？

レポートはリーダー FE にのみ送信されるためです。

## StarRocks のテーブルのフィールドの注釈を確認するにはどうすればよいですか？

`show create table xxx` コマンドを実行します。

## テーブルを作成する際に、NOW() 関数のデフォルト値を指定する方法はありますか？

StarRocks 2.1 以降のバージョンのみが関数のデフォルト値を指定することをサポートしています。StarRocks 2.1 より前のバージョンでは、関数には定数のみを指定できます。

## BE ノードのストレージスペースを解放するにはどうすればよいですか？

`rm -rf` コマンドを使用して `trash` ディレクトリを削除します。スナップショットからデータを復元済みの場合は、`snapshot` ディレクトリを削除できます。

## BE ノードに追加のディスクを追加できますか？

はい。BE の構成項目 `storage_root_path` で指定されたディレクトリにディスクを追加できます。

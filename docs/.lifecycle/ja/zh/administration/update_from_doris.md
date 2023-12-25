---
displayed_sidebar: Chinese
---

# Apache Doris を StarRocks にアップグレードする

この文書では、Apache Doris を StarRocks にアップグレードする方法について説明します。

現在、Apache Doris 0.13.15（それ以前のバージョンを含まない）から StarRocks へのアップグレードのみをサポートしています。Apache Doris 0.13.15 を StarRocks にアップグレードする必要がある場合は、公式の担当者に連絡してソースコードの修正を依頼してください。現在、Apache Doris 0.14 以降のバージョンを StarRocks にアップグレードすることはサポートされていません。

> 注意：
>
> * StarRocks は BE が FE と後方互換性を持つため、**必ず BE ノードを先にアップグレードし、その後で FE ノードをアップグレードしてください**。アップグレードの順序を間違えると、新旧の FE、BE ノードが互換性を持たず、BE ノードがサービスを停止する可能性があります。
> * バージョンを跨いでのアップグレードは慎重に行ってください。バージョンを跨いでのアップグレードが必要な場合は、テスト環境での検証を行い、問題がないことを確認してから本番環境をアップグレードすることをお勧めします。Apache Doris を StarRocks-2.x バージョンにアップグレードする際は、CBO を事前に有効にする必要があります。そのため、まず Apache Doris を StarRocks 1.19.x バージョンにアップグレードする必要があります。1.19.x バージョンのインストールパッケージは[公式ウェブサイト](https://www.mirrorship.cn/zh-CN/download)から入手できます。

## アップグレード環境のチェック

1. MySQL クライアントを使用して、既存のクラスター情報を確認します。

    ```SQL
    SHOW FRONTENDS;
    SHOW BACKENDS;
    SHOW BROKER;
    ```

    FE、BE の数、IP アドレス、バージョンなどの情報、および FE の Leader、Follower、Observer の情報を記録します。

2. インストールパスを確認します。

    以下の例では、元の Apache Doris ディレクトリを `/home/doris/doris/`、新しい StarRocks ディレクトリを `/home/starrocks/starrocks/` としています。誤操作を減らすため、後続の手順では完全なパスを使用します。実際の状況に応じて、例のパス情報を変更してください。

    > 注意：場合によっては、Apache Doris のディレクトリが `/disk1/doris` であることがあります。

3. BE の設定ファイルを確認します。

    **be.conf** ファイル内の `default_rowset_type` 設定項目の値をチェックします：

    * この設定項目の値が `ALPHA` の場合、すべてのデータが segmentV1 形式を使用していることを意味します。この設定項目を BETA に変更し、後続のチェックと変換を行う必要があります。
    * この設定項目の値が `BETA` の場合、データはすでに segmentV2 形式を使用していますが、一部の Tablet または Rowset が segmentV1 形式を使用している可能性があります。後続のチェックと変換が必要です。

4. SQL のテスト状況を確認します。

    ```SQL
    show databases;
    use {one_db};
    show tables;
    show data;
    select count(*) from {one_table};
    ```

## データ形式の変換

変換プロセスには時間がかかる可能性があります。データに segmentV1 形式のデータが多量に存在する場合は、事前に変換操作を行うことをお勧めします。

1. ファイル形式をチェックします。

    a. ファイル形式検出ツールをダウンロードしてインストールします。

    ```bash
    # git clone を使用するか、現在のアドレスから直接ダウンロードすることもできます。
    wget http://Starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/show_segment_status.tar.gz
    tar -zxvf show_segment_status.tar.gz
    ```

    b. インストールパス内の **conf** ファイルを変更し、実際の状況に応じて以下の設定項目を構成します。

    ```conf
    [cluster]
    fe_host =10.0.2.170
    query_port =9030
    user = root
    query_pwd = ****

    # 以下の設定はオプショナルです
    # データベースを一つ選択
    db_names =              // データベース名を記入
    # テーブルを一つ選択
    table_names =           // テーブル名を記入
    # BE を一つ選択。値が 0 の場合はすべての BE を意味します
    be_id = 0
    ```

    c. 設定が完了したら、検出スクリプトを実行してデータ形式をチェックします。

    ```bash
    python show_segment_status.py
    ```

    d. 上記ツールの出力情報 `rowset_count` の二つの値が等しいかどうかをチェックします。数が等しくない場合は、データに segmentV1 形式のテーブルが存在することを意味し、その部分のデータを変換する必要があります。

2. データ形式を変換します。

    a. segmentV1 形式のテーブルのデータ形式を変換します。

    ```SQL
    ALTER TABLE table_name SET ("storage_format" = "v2");
    ```

    b. 変換状態を確認します。`status` フィールドの値が `FINISHED` である場合、形式の変換が完了したことを意味します。

    ```sql
    SHOW ALTER TABLE column;
    ```

    c. データ形式検出ツールを再度実行してデータ形式の状態を確認します。`storage_format` を `V2` に正常に設定したと表示されているにもかかわらず、まだ segmentV1 形式のデータがある場合は、以下の方法でさらにチェックして変換することができます：

      i.   すべてのテーブルを個別にクエリして、テーブルのメタデータリンクを取得します。

    ```sql
    SHOW TABLET FROM table_name;
    ```

      ii.  メタデータリンクを使用して Tablet のメタデータを取得します。

    例：

    ```shell
    wget http://172.26.92.139:8640/api/meta/header/11010/691984191
    ```

      iii. ローカルのメタデータ JSON ファイルを確認し、`rowset_type` の値を確認します。`ALPHA_ROWSET` である場合、そのデータは segmentV1 形式であり、変換が必要です。

      iv.  まだ segmentV1 形式のデータがある場合は、以下の例の方法で変換する必要があります。

      例：

    ```SQL
    ALTER TABLE dwd_user_tradetype_d
    ADD TEMPORARY PARTITION p09
    VALUES [('2020-09-01'), ('2020-10-01')]
    ("replication_num" = "3")
    DISTRIBUTED BY HASH(`dt`, `c`, `city`, `trade_hour`);

    INSERT INTO dwd_user_tradetype_d TEMPORARY partition(p09)
    select * from dwd_user_tradetype_d partition(p202009);

    ALTER TABLE dwd_user_tradetype_d
    REPLACE PARTITION (p202009) WITH TEMPORARY PARTITION (p09);
    ```

## BE ノードのアップグレード

アップグレードは**BE を先に、その後に FE**の順序で行います。

> 注意：BE のアップグレードは一台ずつ行います。現在のマシンのアップグレードが完了した後、一定の時間（1日を推奨）を空けて現在のマシンのアップグレードに問題がないことを確認した後、他のマシンのアップグレードを行います。

1. StarRocks のインストールパッケージをダウンロードして解凍し、インストールパスを **Starrocks** にリネームします。


    ```bash
    cd ~
    tar xzf Starrocks-EE-1.19.6/file/Starrocks-1.19.6.tar.gz
    mv Starrocks-1.19.6/ Starrocks
    ```

2. 新しいBEの **conf/be.conf** に元の **conf/be.conf** の内容を比較してコピーします。

    ```bash
    # 比較して変更とコピー
    vimdiff /home/doris/Starrocks/be/conf/be.conf /home/doris/doris/be/conf/be.conf

    # 以下の設定を新しいBEの設定ファイルにコピーします。元のデータディレクトリの使用を推奨します。
    storage_root_path =     // 元のデータディレクトリ
    ```

3. システムがSupervisorを使用してBEを起動しているかどうかを確認し、BEを停止します。

    ```bash
    # 元のプロセス（palo_be）を確認。
    ps aux | grep palo_be
    # dorisアカウントにSupervisorプロセスがあるか確認。
    ps aux | grep supervisor
    ```

    * SupervisorをデプロイしてBEを起動している場合は、Supervisorを通じてBEを停止するコマンドを送信する必要があります。

    ```bash
    cd /home/doris/doris/be && ./control.sh stop && cd -
    ```

    * Supervisorをデプロイしていない場合は、BEを手動で停止する必要があります。

    ```bash
    sh /home/doris/doris/be/bin/stop_be.sh
    ```

4. BEのプロセス状態を確認し、ノードの `Alive` が `false` であり、`LastStartTime` が最新の時間であることを確認します。

    ```sql
    SHOW BACKENDS;
    ```

    BEプロセスが存在しないことを確認します。

    ```bash
    ps aux | grep be
    ps aux | grep supervisor
    ```

5. 新しいBEノードを起動します。

    ```bash
    sh /home/doris/Starrocks/be/bin/start_be.sh --daemon
    ```

6. アップグレード結果を確認します。

    * 新しいBEプロセスの状態を確認します。

    ```bash
    ps aux | grep be
    ps aux | grep supervisor
    ```

    * MySQLクライアントを通じてノードの `Alive` 状態を確認します。

    ```sql
    SHOW BACKENDS;
    ```

    * **be.out** を確認し、異常なログがないかチェックします。
    * **be.INFO** を確認し、heartbeatが正常かチェックします。
    * **be.WARN** を確認し、異常なログがないかチェックします。

> 注意：2つのBEノードのアップグレードに成功した後、`SHOW FRONTENDS;` を通じて `ReplayedJournalId` が増加しているかどうかを確認し、インポートに問題がないかを検出できます。

## FEノードのアップグレード

BEノードのアップグレードに成功した後、FEノードのアップグレードを続けることができます。

> 注意：FEのアップグレードは**先にObserverをアップグレードし、次にFollowerをアップグレードし、最後にLeaderをアップグレードする**順序に従います。

1. FEのソースコードを変更します（オプション、Apache Doris 0.13.15からのアップグレードが不要な場合は、このステップをスキップできます）。

    a. ソースコードのパッチをダウンロードします。

    ```bash
    wget "http://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/upgrade_from_apache_0.13.15.patch"
    ```

    b. ソースコードにパッチを適用します。

    * Gitを使用してパッチを適用します。

    ```bash
    git apply --reject upgrade_from_apache_0.13.15.patch
    ```

    * ローカルのコードがGit環境にない場合は、パッチの内容に従って手動で適用することもできます。

    c. FEモジュールをコンパイルします。

    ```bash
    ./build.sh --fe --clean
    ```

2. MySQLクライアントを通じて各FEノードのLeaderとFollowerの役割を確認します。

    ```sql
    SHOW FRONTENDS;
    ```

    `IsMaster` が `true` であれば、そのノードはLeaderであり、そうでなければFollowerまたはObserverです。

3. システムがSupervisorを使用してFEを起動しているかどうかを確認し、FEを停止します。

    ```bash
    # 元のプロセスを確認。
    ps aux | grep fe
    # DorisアカウントにSupervisorプロセスがあるか確認します。
    ps aux | grep supervisor
    ```

    * SupervisorをデプロイしてFEを起動している場合は、Supervisorを通じてFEを停止するコマンドを送信する必要があります。

    ```bash
    cd /home/doris/doris/fe && ./control.sh stop && cd -
    ```

    * Supervisorをデプロイしていない場合は、FEを手動で停止する必要があります。

    ```bash
    sh /home/doris/doris/fe/bin/stop_fe.sh
    ```

4. FEのプロセス状態を確認し、ノードの `Alive` が `false` であることを確認します。

    ```sql
    SHOW FRONTENDS;
    ```

    FEプロセスが存在しないことを確認します。

    ```bash
    ps aux | grep fe
    ps aux | grep supervisor
    ```

5. FollowerまたはLeaderをアップグレードする前に、必ずメタデータをバックアップしてください。バックアップのタイミングとしてアップグレードの日付を使用することができます。

    ```bash
    cp -r /home/doris/doris/fe/palo-meta /home/doris/doris/fe/doris-meta
    ```

6. 新しいFEの **conf/fe.conf** に元の **conf/fe.conf** の内容を比較してコピーします。

    ```bash
    # 比較して変更とコピー。
    vimdiff /home/doris/Starrocks/fe/conf/fe.conf /home/doris/doris/fe/conf/fe.conf

    # この行の設定を新しいFEに変更します。元のdorisディレクトリに新しいmetaファイルがあります（後でコピーされます）。
    meta_dir = /home/doris/doris/fe/doris-meta
    # 元のJavaヒープサイズなどの情報を維持します。
    JAVA_OPTS="-Xmx8192m
    ```

7. 新しいFEノードを起動します。

    ```bash
    sh /home/doris/Starrocks/fe/bin/start_fe.sh --daemon
    ```

8. アップグレード結果を確認します。

    * 新しいFEプロセスの状態を確認します。

    ```bash
    ps aux | grep fe
    ps aux | grep supervisor
    ```

    * MySQLクライアントを通じてノードの `Alive` 状態を確認します。

    ```sql
    SHOW FRONTENDS;
    ```

    * **fe.out** または **fe.log** を確認し、異常なログがないかチェックします。
    * **fe.log** が常にUNKNOWN状態で、FollowerやObserverに変わらない場合は、アップグレードに問題があることを示しています。

> 注意：ソースコードを変更してアップグレードした場合は、新しいイメージがメタデータに生成された後（つまり **meta/image** ディレクトリに新しい **image.xxx** ファイルが生成された後）、**fe/lib** パスを新しいインストールパスに戻す必要があります。

## ロールバック計画

Apache Doris からアップグレードした StarRocks は現在ロールバックをサポートしていません。テスト環境での検証が成功した後に本番環境へのアップグレードをお勧めします。

解決できない問題が発生した場合、以下の企業用WeChatを追加してサポートを求めることができます。

![QRコード](../assets/8.3.1.png)

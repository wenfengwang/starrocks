---
displayed_sidebar: "英語"
---

# StarRocks のダウングレード

このトピックでは、StarRocks クラスターをダウングレードする方法について説明します。

StarRocks クラスターをアップグレードした後に例外が発生した場合は、前のバージョンにダウングレードしてクラスターを迅速に復旧させることができます。

## 概要

ダウングレードする前にこのセクションの情報を確認し、推奨されるアクションを実行してください。

### ダウングレードパス

- **パッチバージョンのダウングレードの場合**

  StarRocks クラスターはパッチバージョン間でダウングレードすることができます。たとえば、v2.2.11 から直接 v2.2.6 にダウングレードすることができます。

- **マイナーバージョンのダウングレードの場合**

  互換性と安全性の理由から、StarRocks クラスターを **順次** **一つずつマイナーバージョンを下げて** ダウングレードすることを強く推奨します。たとえば、StarRocks v2.5 クラスターを v2.2 にダウングレードするには、次の順序でダウングレードする必要があります：v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x。

- **メジャーバージョンのダウングレードの場合**

  StarRocks v3.0 クラスターは v2.5.3 以降のバージョンにのみダウングレードすることができます。

  - StarRocks は v3.0 で BDB ライブラリーをアップグレードします。しかし、BDBJE はロールバックできません。ダウングレード後は v3.0 の BDB ライブラリーを使用する必要があります。
  - v3.0 にアップグレードすると、新しい RBAC 権限システムがデフォルトで使用されます。ダウングレード後も RBAC 権限システムを使用することができます。

### ダウングレード手順

StarRocks のダウングレード手順は、[アップグレード手順](../deployment/upgrade.md#upgrade-procedure)の逆の順序です。したがって、**FE を最初にダウングレード** **し、次に BE と CN** をダウングレードする必要があります。彼らを誤った順序でダウングレードすると、FE と BE/CN の間で互換性が失われ、サービスがクラッシュする可能性があります。FE ノードの場合は、Leader FE ノードをダウングレードする前にすべての Follower FE ノードを先にダウングレードする必要があります。

## 開始する前に

準備中に、マイナーバージョンまたはメジャーバージョンをダウングレードする場合は互換性設定を実行する必要があります。また、クラスターのすべてのノードをダウングレードする前に、FE または BE のいずれかでダウングレードの可用性テストを実行する必要があります。

### 互換性設定の実行

StarRocks クラスターを以前のマイナーバージョンまたはメジャーバージョンにダウングレードする場合、互換性設定を実行する必要があります。一般的な互換性設定に加えて、ダウングレード元の StarRocks クラスターバージョンによって詳細な設定が異なります。

- **一般的な互換性設定**

StarRocks クラスターをダウングレードする前に、タブレットクローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

ダウングレード後、すべての BE ノードの状態が `Alive` になった場合に限り、タブレットクローンを再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.2 以降のバージョンからダウングレードする場合**

FE 設定項目 `ignore_unknown_log_id` を `true` に設定します。これは静的なパラメータなので、FE 設定ファイル **fe.conf** に変更を加えてノードを再起動し、変更を有効にする必要があります。ダウングレードと最初のチェックポイントが完了した後、この設定を `false` に戻し、ノードを再起動できます。

- **FQDN アクセスを有効化している場合**

もしあなたが FQDN アクセスを有効化している場合（v2.4 からサポート）、v2.4 より前のバージョンにダウングレードする必要がある場合は、ダウングレード前に IP アドレスアクセスに切り替える必要があります。詳細な手順については、[FQDN のロールバック](../administration/enable_fqdn.md#rollback) を参照してください。

## FE のダウングレード

互換性設定と可用性テストの後、FE ノードをダウングレードすることができます。まず Follower FE ノードをダウングレードし、その後 Leader FE ノードをダウングレードします。

1. FE ノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir> を FE ノードの展開ディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. **bin**、**lib**、および **spark-dpp** の元の展開ファイルを、以前のバージョンのものと置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

   > **注意**
   >
   > StarRocks v3.0 を v2.5 にダウングレードする場合、展開ファイルを置き換えた後に次の手順に従ってください。
   >
   > 1. v3.0 展開の **fe/lib/starrocks-bdb-je-18.3.13.jar** ファイルを v2.5 展開の **fe/lib** ディレクトリにコピーします。
   > 2. **fe/lib/je-7.\*.jar** ファイルを削除します。

3. FE ノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. FE ノードが正常に起動されたかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 上記の手順を繰り返して他の Follower FE ノードをダウングレードし、最後に Leader FE ノードをダウングレードします。

   > **注意**
   >
   > StarRocks v3.0 を v2.5 にダウングレードする場合、ダウングレード後に次の手順に従ってください。
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべての Follower FE に同期されるのを待ちます。
   >
   > このコマンドを実行しない場合、ダウングレード操作のいくつかが失敗する可能性があります。ALTER SYSTEM CREATE IMAGE は v2.5.3 以降でサポートされています。

## BE のダウングレード

FE ノードをダウングレードした後、クラスター内の BE ノードをダウングレードできます。

1. BE ノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir> を BE ノードの展開ディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. **bin** と **lib** の元の展開ファイルを、以前のバージョンのものと置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. BE ノードを起動します。

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. BE ノードが正常に起動されたかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して他の BE ノードをダウングレードします。

## CN のダウングレード

1. CN ノードの作業ディレクトリに移動し、ノードを穏やかに停止します。

   ```Bash
   # <cn_dir> を CN ノードの展開ディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. **bin** と **lib** の元の展開ファイルを、以前のバージョンのものと置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. CN ノードを起動します。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. CN ノードが正常に起動されたかどうかを確認します。

   ```Bash
   ps aux | grep  starrocks_be
   ```

5. 上記の手順を繰り返して他の CN ノードをダウングレードします。
---
displayed_sidebar: Chinese
---

# StarRocks のダウングレード

この文書では、StarRocks クラスターをダウングレードする方法について説明します。

StarRocks クラスターをアップグレードした後に異常が発生した場合、以前のバージョンにダウングレードしてクラスターを迅速に復旧することができます。

## 概要

ダウングレードする前に、このセクションの情報を確認してください。文中で推奨される手順に従ってクラスターをダウングレードすることをお勧めします。

### ダウングレードパス

- **マイナーバージョンダウングレード**

  StarRocks クラスターをマイナーバージョン間でダウングレードすることができます。例えば、v2.2.11 から直接 v2.2.6 にダウングレードすることが可能です。

- **メジャーバージョンダウングレード**

  互換性とセキュリティの理由から、StarRocks クラスターを**メジャーバージョンごとに段階的にダウングレードすることを強く推奨します**。例えば、StarRocks v2.5 クラスターを v2.2 にダウングレードするには、次の順序でダウングレードする必要があります：v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x。

- **メジャーバージョンダウングレード**

  - v1.19 へのクロスバージョンダウングレードはできません。まず v2.0 にダウングレードする必要があります。
  - クラスターを v3.0 から v2.5.3 以上のバージョンにダウングレードすることのみが可能です。
    - StarRocks は v3.0 で BDB ライブラリをアップグレードしました。BDB JE はロールバックできないため、ダウングレード後も v3.0 の BDB ライブラリを使用し続ける必要があります。
    - v3.0 にアップグレードした後、クラスターはデフォルトで新しい RBAC 権限システムを使用します。ダウングレード後も RBAC 権限システムを使用することになります。

### ダウングレードプロセス

StarRocks のダウングレードプロセスは [アップグレードプロセス](../deployment/upgrade.md#アップグレードプロセス) とは逆です。したがって、**まず FE をダウングレードし、次に BE と CN をダウングレードする必要があります**。誤ったダウングレード順序は FE と BE/CN の互換性がなくなり、サービスがクラッシュする可能性があります。FE ノードについては、すべての Follower FE ノードを先にダウングレードし、最後に Leader FE ノードをダウングレードする必要があります。

## 準備作業

準備中に、メジャーバージョンまたはメジャーバージョンのダウングレードが必要な場合は、互換性のある設定を行う必要があります。クラスターのすべてのノードを完全にダウングレードする前に、FE と BE のノードの一つでダウングレードの正確性をテストする必要があります。

### 互換性設定

メジャーバージョンまたはメジャーバージョンのダウングレードを行う場合は、互換性設定が必要です。一般的な互換性設定に加えて、ダウングレード前のバージョンに応じた具体的な設定も必要です。

- **一般的な互換性設定**

ダウングレード前に、Tablet Clone を無効にしてください。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

ダウングレードが完了し、すべての BE ノードの状態が `Alive` になった後、Tablet Clone を再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.2 以降のバージョンからダウングレードする場合**

FE の設定項目 `ignore_unknown_log_id` を `true` に設定してください。この設定項目は静的パラメータであるため、FE の設定ファイル **fe.conf** で変更し、変更後にノードを再起動して有効にする必要があります。ダウングレードが終了し、最初の Checkpoint が完了した後、`false` にリセットしてノードを再起動することができます。

- **FQDN アクセスを有効にした場合**

FQDN アクセス（v2.4 からサポート）を有効にしていて、v2.4 以前のバージョンにダウングレードする必要がある場合は、ダウングレード前に IP アドレスアクセスに切り替える必要があります。詳細については、[FQDN のロールバック](../administration/enable_fqdn.md#FQDNのロールバック) を参照してください。

## FE のダウングレード

ダウングレードの正確性をテストした後、FE ノードを最初にダウングレードすることができます。まず Follower FE ノードをダウングレードし、次に Leader FE ノードをダウングレードする必要があります。

1. FE ノードの作業ディレクトリに入り、そのノードを停止します。

   ```Bash
   # <fe_dir> を FE ノードのデプロイディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. デプロイファイルの既存のパス **bin**、**lib**、および **spark-dpp** を古いバージョンのデプロイファイルに置き換えます。

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
   > StarRocks v3.0 を v2.5 にダウングレードする場合は、デプロイファイルを置き換えた後、以下の手順を実行する必要があります：
   >
   > 1. v3.0 のデプロイファイルにある **fe/lib/starrocks-bdb-je-18.3.13.jar** を v2.5 のデプロイファイルの **fe/lib** パスにコピーします。
   > 2. **fe/lib/je-7.\*.jar** ファイルを削除します。

3. その FE ノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. ノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 上記の手順を繰り返して他の Follower FE ノードをダウングレードし、最後に Leader FE ノードをダウングレードします。

   > **注意**
   >
   > StarRocks v3.0 を v2.5 にダウングレードする場合は、ダウングレードが完了した後、以下の手順を実行する必要があります：
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を実行して新しいメタデータスナップショットファイルを作成します。
   > 2. 他の FE ノードにメタデータスナップショットファイルが同期されるのを待ちます。
   >
   > このコマンドを実行しないと、一部のダウングレード操作が失敗する可能性があります。ALTER SYSTEM CREATE IMAGE コマンドは v2.5.3 以降のバージョンでのみサポートされています。

## BE のダウングレード

すべての FE ノードをダウングレードした後、BE ノードのダウングレードを続けることができます。

1. BE ノードの作業ディレクトリに入り、そのノードを停止します。

   ```Bash
   # <be_dir> を BE ノードのデプロイディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. デプロイファイルの既存のパス **bin** と **lib** を古いバージョンのデプロイファイルに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. その BE ノードを起動します。

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. ノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して他の BE ノードをダウングレードします。

## CN のダウングレード

1. CN ノードの作業ディレクトリに入り、そのノードを優雅に停止します。

   ```Bash
   # <cn_dir> を CN ノードのデプロイディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. デプロイファイルの既存のパス **bin** と **lib** を新しいバージョンのデプロイファイルに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. その CN ノードを起動します。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. ノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して他の CN ノードをダウングレードします。

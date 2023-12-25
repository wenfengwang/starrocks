---
displayed_sidebar: English
---

# StarRocksをダウングレード

このトピックでは、StarRocks クラスターをダウングレードする方法について説明します。

StarRocks クラスタのアップグレード後に例外が発生した場合は、以前のバージョンにダウングレードして、クラスタを迅速に復旧できます。

## 概要

ダウングレードする前に、このセクションの情報を確認してください。推奨されるアクションを実行します。

### ダウングレード パス

- **パッチバージョンのダウングレードの場合**

  StarRocksクラスタは、パッチバージョン間で、たとえばv2.2.11からv2.2.6に直接ダウングレードできます。

- **マイナーバージョンのダウングレードの場合**

  互換性と安全上の理由から、StarRocksクラスターを**1つのマイナーバージョンから別のマイナーバージョンに連続して**ダウングレードすることを強くお勧めします。たとえば、StarRocks v2.5 クラスタを v2.2 にダウングレードするには、v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x の順序でダウングレードする必要があります。

- **メジャーバージョンのダウングレードの場合**

  StarRocks v3.0 クラスターは、v2.5.3 以降のバージョンにのみダウングレードできます。

  - StarRocksは、v3.0でBDBライブラリをアップグレードします。ただし、BDBJE をロールバックすることはできません。ダウングレード後は、v3.0のBDBライブラリを使用する必要があります。
  - 新しい RBAC 権限システムは、v3.0 へのアップグレード後にデフォルトで使用されます。RBAC 特権システムは、ダウングレード後にのみ使用できます。

### ダウングレード手順

StarRocksのダウングレード手順は、[アップグレード手順](../deployment/upgrade.md#upgrade-procedure)の逆の順序です。したがって、最初に**FE**を**ダウングレード**し、次に**BE**と**CN**をダウングレードする必要があります。間違った順序でダウングレードすると、FE と BE/CN の間に互換性がなくなり、サービスがクラッシュする可能性があります。FEノードの場合、リーダーFEノードをダウングレードする前に、まずすべてのFollower FEノードをダウングレードする必要があります。

## 始める前に

準備中に、マイナー バージョンまたはメジャー バージョンのダウングレードを行う場合は、互換性構成を実行する必要があります。また、クラスタ内のすべてのノードをダウングレードする前に、FE または BE の 1 つでダウングレード可用性テストを実行する必要があります。

### 互換性構成の実行

StarRocksクラスタを以前のマイナーバージョンまたはメジャーバージョンにダウングレードする場合は、互換性設定を実行する必要があります。ユニバーサル互換性設定に加えて、詳細な設定は、ダウングレード元のStarRocksクラスタのバージョンによって異なります。

- **ユニバーサル互換性構成**

StarRocksクラスタをダウングレードする前に、タブレットクローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

ダウングレード後、すべてのBEノードのステータスが`Alive`になったら、タブレットクローンを再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.2 以降のバージョンからダウングレードする場合**

FE コンフィギュレーション項目`ignore_unknown_log_id`を`true`に設定します。これは静的パラメータであるため、FEコンフィギュレーションファイル**fe.conf**で変更し、ノードを再起動して変更を有効にする必要があります。ダウングレードと最初のチェックポイントが完了したら、`ignore_unknown_log_id`を`false`にリセットしてノードを再起動できます。

- **FQDN アクセスを有効にしている場合**

FQDN アクセス(v2.4 以降でサポート)を有効にしていて、v2.4 より前のバージョンにダウングレードする必要がある場合は、ダウングレードする前に IP アドレス アクセスに切り替える必要があります。詳細な手順については、「[FQDN のロールバック](../administration/enable_fqdn.md#rollback)」を参照してください。

## FEのダウングレード

互換性設定と可用性テストの後、FE ノードをダウングレードできます。最初にFollower FEノードをダウングレードし、次にLeader FEノードをダウングレードする必要があります。

1. FEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir>をFEノードのデプロイメントディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. **bin**、**lib**、および**spark-dpp**の下にある元のデプロイメントファイルを以前のバージョンのファイルに置き換えます。

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
   > StarRocks v3.0 を v2.5 にダウングレードする場合は、デプロイメントファイルを置き換えた後、次の手順に従う必要があります。
   >
   > 1. v3.0 デプロイメントのファイル**starrocks-bdb-je-18.3.13.jar**をv2.5 デプロイメントのディレクトリ**fe/lib**にコピーします。
   > 2. ファイル**je-7.\*.jar**を削除します。

3. FEノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. FEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 上記の手順を繰り返して、他のFollower FEノードをダウングレードし、最後にLeader FEノードをダウングレードします。

   > **注意**
   >
   > StarRocks v3.0 を v2.5 にダウングレードする場合は、ダウングレード後に次の手順に従う必要があります。
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべてのFollower FEに同期されるのを待ちます。
   >
   > このコマンドを実行しないと、一部のダウングレード操作が失敗する可能性があります。ALTER SYSTEM CREATE IMAGEはv2.5.3以降でサポートされています。

## BEのダウングレード

FE ノードをダウングレードしたら、クラスタ内の BE ノードをダウングレードできます。

1. BEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir>をBEノードのデプロイメントディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. **bin**と**lib**の下にある元のデプロイメントファイルを以前のバージョンのものに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. BEノードを起動します。

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. BEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して、他のBEノードをダウングレードします。

## CNのダウングレード

1. CNノードの作業ディレクトリに移動し、ノードを正常に停止します。

   ```Bash
   # <cn_dir>をCNノードのデプロイメントディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. **bin**と**lib**の下にある元のデプロイメントファイルを以前のバージョンのものに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. CNノードを起動します。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. CNノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して、他のCNノードをダウングレードします。

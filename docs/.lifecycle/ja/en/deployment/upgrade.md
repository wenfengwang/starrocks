---
displayed_sidebar: English
---

# StarRocksのアップグレード

このトピックでは、StarRocksクラスターをアップグレードする方法について説明します。

## 概要

アップグレードする前に、このセクションの情報をレビューし、推奨されるアクションを実行してください。

### StarRocksのバージョン

StarRocksのバージョンは、`Major.Minor.Patch`の形式で表されます。例えば、`2.5.4`です。最初の数字はStarRocksのメジャーバージョンを表し、2番目の数字はマイナーバージョンを表し、3番目の数字はパッチバージョンを表します。StarRocksは、特定のバージョンに対して長期サポート（LTS）を提供しています。そのサポート期間は半年以上続きます。

| **StarRocksバージョン** | **LTSバージョンですか** |
| --------------------- | ---------------------- |
| v2.0.x                | いいえ                     |
| v2.1.x                | いいえ                     |
| v2.2.x                | いいえ                     |
| v2.3.x                | いいえ                     |
| v2.4.x                | いいえ                     |
| v2.5.x                | はい                    |
| v3.0.x                | いいえ                     |
| v3.1.x                | いいえ                     |

### アップグレードパス

- **パッチバージョンアップグレードの場合**

  StarRocksクラスターは、パッチバージョン間でアップグレード可能です。例えば、v2.2.6からv2.2.11へ直接アップグレードできます。

- **マイナーバージョンアップグレードの場合**

  StarRocks v2.0以降、StarRocksクラスターはマイナーバージョン間でアップグレード可能です。例えば、v2.2.xからv2.5.xへ直接アップグレードできます。しかし、互換性と安全性の理由から、StarRocksクラスターを**連続して一つのマイナーバージョンから次のマイナーバージョンへ**アップグレードすることを強く推奨します。例えば、StarRocks v2.2クラスターをv2.5にアップグレードする場合、次の順序でアップグレードする必要があります：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **メジャーバージョンアップグレードの場合**

  StarRocksクラスターをv3.0にアップグレードするには、先にv2.5にアップグレードする必要があります。

### アップグレード手順

StarRocksは**ローリングアップグレード**をサポートしており、サービスを停止せずにクラスターをアップグレードできます。設計上、BEとCNはFEとの下位互換性があります。したがって、クラスターがアップグレード中に正常に稼働するためには、**最初にBEとCNをアップグレードし、その後でFE**をアップグレードする必要があります。逆の順序でアップグレードすると、FEとBE/CNの間で互換性がなくなり、サービスがクラッシュする可能性があります。FEノードについては、リーダーFEノードをアップグレードする前に、すべてのフォロワーFEノードを先にアップグレードする必要があります。

## 始める前に

準備中に、マイナーバージョンアップグレードまたはメジャーバージョンアップグレードを行う場合は、互換性設定を実行する必要があります。また、クラスタ内のすべてのノードをアップグレードする前に、FEとBEのいずれかでアップグレード可用性テストを実施する必要があります。

### 互換性設定の実施

StarRocksクラスターを新しいマイナーバージョンまたはメジャーバージョンにアップグレードする場合は、互換性設定を実施する必要があります。ユニバーサル互換性設定に加えて、アップグレード元のStarRocksクラスターのバージョンに応じて、詳細な設定が異なります。

- **ユニバーサル互換性設定**

StarRocksクラスターをアップグレードする前に、タブレットクローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

アップグレード後、すべてのBEノードのステータスが`Alive`であれば、タブレットクローンを再度有効にできます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.0からそれ以降のバージョンにアップグレードする場合**

StarRocks v2.0クラスターをアップグレードする前に、以下のBE設定とシステム変数を設定する必要があります。

1. BE設定項目`vector_chunk_size`を変更している場合は、アップグレード前にそれを`4096`に設定する必要があります。これは静的パラメータであるため、BE設定ファイル**be.conf**で変更し、ノードを再起動して変更を有効にする必要があります。
2. システム変数`batch_size`をグローバルに`4096`以下に設定します。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## BEのアップグレード

アップグレード可用性テストに合格したら、まずクラスタ内のBEノードをアップグレードします。

1. BEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir>をBEノードのデプロイメントディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. **bin**と**lib**の下にある元のデプロイメントファイルを新しいバージョンのものに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib .
   cp -r /tmp/StarRocks-x.x.x/be/bin .
   ```

3. BEノードを起動します。

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. BEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して、他のBEノードをアップグレードします。

## CNのアップグレード

1. CNノードの作業ディレクトリに移動し、ノードを正常に停止します。

   ```Bash
   # <cn_dir>をCNノードのデプロイメントディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. **bin**と**lib**の下にある元のデプロイメントファイルを新しいバージョンのものに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib .
   cp -r /tmp/StarRocks-x.x.x/be/bin .
   ```

3. CNノードを起動します。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. CNノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 上記の手順を繰り返して、他のCNノードをアップグレードします。

## FEのアップグレード

すべてのBEノードとCNノードをアップグレードした後、FEノードをアップグレードします。最初にフォロワーFEノードをアップグレードし、次にリーダーFEノードをアップグレードする必要があります。

1. FEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir>をFEノードのデプロイメントディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. **bin**、**lib**、および**spark-dpp**の下にある元のデプロイメントファイルを新しいバージョンのものに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib .
   cp -r /tmp/StarRocks-x.x.x/fe/bin .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp .
   ```

3. FEノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. FEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 上記の手順を繰り返して、他のフォロワーFEノードをアップグレードし、最後にリーダーFEノードをアップグレードします。

   > **注意**
   >
   > StarRocksクラスターをv2.5からv3.0にアップグレードした後にダウングレードし、再度v3.0にアップグレードする場合、一部のフォロワーFEのメタデータアップグレード失敗を回避するために、以下の手順に従う必要があります：
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべてのフォロワーFEに同期されるのを待ちます。
   >
   > イメージファイルが同期されたかどうかは、リーダーFEのログファイル**fe.log**を確認することでわかります。"push image.* from subdir [] to other nodes. Totally xx nodes, push successful xx nodes"というログがあれば、イメージファイルが正常に同期されたことを示します。

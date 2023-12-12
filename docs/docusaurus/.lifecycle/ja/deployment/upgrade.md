---
displayed_sidebar: "Japanese"
---

# StarRocksのアップグレード

このトピックでは、StarRocksクラスターのアップグレード方法について説明します。

## 概要

アップグレード前に、このセクションの情報を確認し、推奨されるアクションを実行してください。

### StarRocksのバージョン

StarRocksのバージョンは、**Major.Minor.Patch**という形式の3つの数字で表されます。たとえば、`2.5.4`です。最初の数字はStarRocksのメジャーバージョンを、2番目の数字はマイナーバージョンを、3番目の数字はパッチバージョンを表します。StarRocksは特定のバージョンに対して長期サポート（LTS）を提供しており、そのサポート期間は半年以上続きます。

| **StarRocksバージョン** | **LTSバージョンかどうか** |
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

- **パッチバージョンのアップグレード**

  StarRocksクラスターをパッチバージョン間でアップグレードすることができます。たとえば、v2.2.6から直接v2.2.11にアップグレードすることができます。

- **マイナーバージョンのアップグレード**

  StarRocks v2.0以降、StarRocksクラスターをマイナーバージョン間でアップグレードすることができます。たとえば、v2.2.xから直接v2.5.xにアップグレードすることができます。ただし、互換性と安全性の観点から、StarRocksクラスターを**連続して1つのマイナーバージョンから別のマイナーバージョンにアップグレードすることを強くお勧めします**。たとえば、StarRocks v2.2クラスターをv2.5にアップグレードするには、次の順序でアップグレードする必要があります：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **メジャーバージョンのアップグレード**

  StarRocksクラスターをv3.0にアップグレードするには、まずv2.5にアップグレードする必要があります。

### アップグレード手順

StarRocksは**ローリングアップグレード**をサポートしており、サービスを停止することなくクラスターをアップグレードすることができます。設計上、BE（バックエンド）とCN（クエリノード）はFE（フロントエンド）との後方互換性があります。したがって、クラスターを適切に実行しながらアップグレードするには、**最初にBEとCNをアップグレードし、その後FEをアップグレードする必要があります**。逆の順序でアップグレードすると、FEとBE/CNの間で互換性がなくなり、それによってサービスがクラッシュする可能性があります。FEノードの場合、リーダーFEノードをアップグレードする前に、まずすべてのフォロワーFEノードをアップグレードする必要があります。

## 開始前に

準備段階では、マイナーやメジャーバージョンのアップグレードを行う場合に、互換性の構成を実行する必要があります。また、クラスター全体のノードをアップグレードする前に、1つのFEおよびBEでアップグレード可用性テストを実行する必要があります。

### 互換性の構成を実行する

StarRocksクラスターを後のマイナーやメジャーバージョンにアップグレードする場合は、互換性の構成を実行する必要があります。一般的な互換性の構成に加えて、詳細な構成はアップグレード元のStarRocksクラスターバージョンによって異なります。

- **一般的な互換性の構成**

StarRocksクラスターをアップグレードする前に、タブレットのクローンを無効にする必要があります。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

アップグレード後、すべてのBEノードのステータスが`Alive`であれば、タブレットのクローンを再度有効にできます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.0から後のバージョンにアップグレードする場合**

StarRocks v2.0クラスターをアップグレードする前に、次のBE構成項目およびシステム変数を設定する必要があります。

1. BE構成項目`vector_chunk_size`を変更した場合は、アップグレード前にそれを`4096`に設定する必要があります。これは静的パラメータであるため、BE構成ファイル**be.conf**で変更を行い、変更を有効にするためにノードを再起動する必要があります。
2. システム変数`batch_size`をグローバルに`4096`以下に設定してください。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## BEのアップグレード

アップグレード可用性テストをパスしたら、まずクラスター内のBEノードをアップグレードできます。

1. BEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <be_dir>をBEノードの展開ディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 新しいバージョンのデプロイメントファイルで元の**bin**および**lib**を置き換えます。

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

5. 他のBEノードをアップグレードするために上記の手順を繰り返します。

## CNのアップグレード

1. CNノードの作業ディレクトリに移動し、ノードを正常に停止します。

   ```Bash
   # <cn_dir>をCNノードの展開ディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 新しいバージョンのデプロイメントファイルで元の**bin**および**lib**を置き換えます。

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

5. 他のCNノードをアップグレードするために上記の手順を繰り返します。

## FEのアップグレード

すべてのBEおよびCNノードをアップグレードした後、FEノードをアップグレードできます。まず、フォロワーFEノードをアップグレードし、その後にリーダーFEノードをアップグレードする必要があります。

1. FEノードの作業ディレクトリに移動し、ノードを停止します。

   ```Bash
   # <fe_dir>をFEノードの展開ディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 新しいバージョンのデプロイメントファイルで元の**bin**、**lib**、および**spark-dpp**を置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

3. FEノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. FEノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 他のフォロワーFEノードをアップグレードし、最後にリーダーFEノードをアップグレードします。

   > **注意**
   >
   > v2.5からv3.0にアップグレードした後にStarRocksクラスターをダウングレードし、再度v3.0にアップグレードする場合、Follower FEの一部にメタデータのアップグレードの失敗を避けるために、次の手順に従う必要があります：
   >
   > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成します。
   > 2. 新しいイメージがすべてのFollower FEに同期されるのを待ちます。
   >
   > 新しいイメージファイルが同期されたかどうかは、リーダーFEのログファイル**fe.log**を確認することで確認できます。"push image.* from subdir [] to other nodes. totally xx nodes, push successful xx nodes"というログがあれば、イメージファイルが正常に同期されたことを示します。
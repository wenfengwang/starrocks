---
displayed_sidebar: Chinese
---

# StarRocks のアップグレード

この文書では、StarRocks クラスターのアップグレード方法について説明します。

## 概要

アップグレード前に、このセクションの情報を確認してください。文中の推奨される手順に従ってクラスターをアップグレードすることをお勧めします。

### StarRocks のバージョン

StarRocks のバージョン番号は、**Major.Minor.Patch** の形式の3つの数字で表され、例えば `2.5.4` です。最初の数字は StarRocks のメジャーバージョンを表し、2番目の数字はマイナーバージョンを表し、3番目の数字はパッチバージョンを表します。現在、StarRocks は一部のバージョンに対して長期サポート（Long-time Support、LTS）を提供しており、そのメンテナンス期間は半年以上です。

| **StarRocks のバージョン** | **LTS バージョンですか？** |
| ------------------ | ------------------- |
| v1.19.x            | いいえ              |
| v2.0.x             | いいえ              |
| v2.1.x             | いいえ              |
| v2.2.x             | いいえ              |
| v2.3.x             | いいえ              |
| v2.4.x             | いいえ              |
| v2.5.x             | はい                |
| v3.0.x             | いいえ              |
| v3.1.x             | いいえ              |

### アップグレードパス

- **パッチバージョンのアップグレード**

  小バージョン間で StarRocks クラスターをアップグレードすることができます。例えば、v2.2.6 から v2.2.11 へ直接アップグレードすることができます。

- **マイナーバージョンのアップグレード**

  StarRocks v2.0 から、マイナーバージョン間で StarRocks クラスターをアップグレードすることができます。例えば、v2.2.x から v2.5.x へ直接アップグレードすることができます。しかし、互換性とセキュリティの理由から、**マイナーバージョンを段階的にアップグレードすることを強く推奨します**。例えば、StarRocks v2.2 クラスターを v2.5 にアップグレードする場合、次の順序でアップグレードする必要があります：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **メジャーバージョンのアップグレード**

  - v1.19 から v2.0 へのアップグレードが必要です。
  - v2.5 から v3.0 へのアップグレードが必要です。

### アップグレードプロセス

StarRocks は**ローリングアップグレード**をサポートしており、サービスを停止することなくクラスターをアップグレードすることができます。設計により、BE と CN は FE との後方互換性があります。したがって、**BE と CN を先にアップグレードし、その後で FE をアップグレードする**必要があります。これにより、アップグレード中もクラスターが正常に動作することができます。誤ったアップグレード順序は、FE と BE/CN の互換性がなくなり、サービスがクラッシュする原因となる可能性があります。FE ノードについては、すべての Follower FE ノードを先にアップグレードし、最後に Leader FE ノードをアップグレードする必要があります。

## 準備作業

準備プロセス中に、マイナーバージョンまたはメジャーバージョンのアップグレードが必要な場合は、互換性の設定が必要です。クラスターのすべてのノードを完全にアップグレードする前に、1つの FE と BE ノードでアップグレードの正確性をテストする必要があります。

### 互換性の設定

マイナーバージョンまたはメジャーバージョンのアップグレードが必要な場合は、互換性の設定が必要です。一般的な互換性の設定に加えて、アップグレード前のバージョンに基づいた具体的な設定が必要です。

- **一般的な互換性の設定**

アップグレード前に、Tablet Clone を無効にしてください。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

アップグレードが完了し、すべての BE ノードの状態が `Alive` になった後、Tablet Clone を再度有効にすることができます。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **v2.0 からのアップグレード**

v2.0 バージョンから他のマイナーバージョンへのアップグレード時には、以下の BE 設定項目とシステム変数を設定する必要があります。

1. `vector_chunk_size` の BE 設定項目を以前に変更したことがある場合は、アップグレード前に `4096` に設定する必要があります。この設定項目は静的パラメータであるため、BE の設定ファイル **be.conf** で変更し、変更後にノードを再起動して変更を有効にする必要があります。
2. システム変数 `batch_size` を `4096` 以下の値にグローバル設定します。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## BE のアップグレード

アップグレードの正確性をテストした後、クラスター内の BE ノードを最初にアップグレードすることができます。

1. BE ノードの作業ディレクトリに移動し、そのノードを停止します。

   ```Bash
   # <be_dir> を BE ノードのデプロイメントディレクトリに置き換えてください。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. **bin** および **lib** の既存のデプロイメントファイルを新しいバージョンのファイルに置き換えます。

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

5. 上記の手順を繰り返して、他の BE ノードをアップグレードします。

## CN のアップグレード

1. CN ノードの作業ディレクトリに移動し、そのノードを優雅に停止します。

   ```Bash
   # <cn_dir> を CN ノードのデプロイメントディレクトリに置き換えてください。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. **bin** および **lib** の既存のデプロイメントファイルを新しいバージョンのファイルに置き換えます。

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

5. 上記の手順を繰り返して、他の CN ノードをアップグレードします。

## FE のアップグレード

すべての BE および CN ノードをアップグレードした後、FE ノードのアップグレードを続けることができます。まず Follower FE ノードをアップグレードし、次に Leader FE ノードをアップグレードする必要があります。

1. FE ノードの作業ディレクトリに移動し、そのノードを停止します。

   ```Bash
   # <fe_dir> を FE ノードのデプロイメントディレクトリに置き換えてください。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. **bin**、**lib**、および **spark-dpp** の既存のデプロイメントファイルを新しいバージョンのファイルに置き換えます。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

3. その FE ノードを起動します。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. ノードが正常に起動したかどうかを確認します。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 上記の手順を繰り返して、他の Follower FE ノードをアップグレードし、最後に Leader FE ノードをアップグレードします。

  > **注意**
  >
  > v2.5 から v3.0 へのアップグレード後にロールバックを行い、その後再び v3.0 へのアップグレードを行う場合、一部の Follower FE ノードでメタデータのアップグレードが失敗するのを避けるために、アップグレード完了後に以下の手順を実行する必要があります：
  >
  > 1. [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を実行して、新しいメタデータのスナップショットファイルを作成します。
  > 2. そのメタデータのスナップショットファイルが他の FE ノードに同期されるのを待ちます。
  >
  > Leader FE ノードのログファイル **fe.log** を確認することで、メタデータのスナップショットファイルが完全にプッシュされたかどうかを確認できます。ログに「push image.* from subdir [] to other nodes. totally xx nodes, push successed xx nodes」と表示されていれば、スナップショットファイルのプッシュが完了したことを意味します。

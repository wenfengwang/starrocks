---
displayed_sidebar: Chinese
---

# StarRocks のスケールアウトとスケールイン

この文書では、StarRocks クラスタのスケールアウトとスケールインの方法について説明します。

## FE クラスタのスケールアウトとスケールイン

StarRocks の FE ノードは Follower ノードと Observer ノードに分けられます。Follower ノードは選挙投票と書き込みに参加し、Observer ノードはログの同期と読み取り性能の拡張にのみ使用されます。

> 注意：
>
> * すべての FE ノードの `http_port` は同じでなければなりません。
> * Follower FE ノード（Leader ノードを含む）の数は奇数が推奨されます。3つの Follower ノードをデプロイして、高可用性（HA）モードを構成することをお勧めします。
> * FE クラスタが既に高可用性モード（つまり、1つの Leader ノードと2つの Follower ノードを含む）である場合、Observer ノードを追加して FE の読み取りサービス能力を拡張することをお勧めします。
> * 通常、1つの FE ノードは10から20台の BE ノードに対応できます。FE クラスタのノード数は10以下に抑えることをお勧めします。通常、3つの FE ノードでほとんどの要件を満たすことができます。

### FE クラスタのスケールアウト

新しい FE ノードをデプロイして起動します。詳細なデプロイ方法は [StarRocks のデプロイ](../deployment/deploy_manually.md) を参照してください。

```bash
bin/start_fe.sh --helper "fe_leader_host:edit_log_port" --daemon
```

`fe_leader_host`：Leader FE ノードの IP アドレスです。

FE クラスタをスケールアウトします。新しいノードを Follower または Observer ノードとして設定できます。

* 新しいノードを Follower ノードとして設定する。

```sql
ALTER SYSTEM ADD follower "fe_host:edit_log_port";
```

* 新しいノードを Observer ノードとして設定する。

```sql
ALTER SYSTEM ADD observer "fe_host:edit_log_port";
```

完了したら、ノード情報を確認してスケールアウトが成功したかどうかを検証できます。

```sql
SHOW PROC '/frontends';
```

### FE クラスタのスケールイン

Follower または Observer ノードを削除できます。

* Follower ノードを削除する。

```sql
ALTER SYSTEM DROP follower "fe_host:edit_log_port";
```

* Observer ノードを削除する。

```sql
ALTER SYSTEM DROP observer "fe_host:edit_log_port";
```

完了したら、ノード情報を確認してスケールインが成功したかどうかを検証できます。

```sql
SHOW PROC '/frontends';
```

## BE クラスタのスケールアウトとスケールイン

BE クラスタのスケールアウトとスケールインが成功すると、StarRocks は自動的に負荷状況に基づいてデータのバランスを取り、この期間中にシステムは正常に稼働します。

### BE クラスタのスケールアウト

新しい BE ノードをデプロイして起動します。詳細なデプロイ方法は [StarRocks のデプロイ](../deployment/deploy_manually.md) を参照してください。

```bash
bin/start_be.sh --daemon
```

BE クラスタをスケールアウトします。

```sql
ALTER SYSTEM ADD backend 'be_host:be_heartbeat_service_port';
```

完了したら、ノード情報を確認してスケールアウトが成功したかどうかを検証できます。

```sql
SHOW PROC '/backends';
```

### BE クラスタのスケールイン

DROP または DECOMMISSION の方法で BE クラスタをスケールインできます。

DROP は BE ノードを即座に削除し、失われたレプリカは FE によって補充されます。一方、DECOMMISSION はレプリカが補充された後に BE ノードを削除します。BE クラスタのスケールインには DECOMMISSION 方法をお勧めします。

* DECOMMISSION 方法で BE クラスタをスケールインする。

```sql
ALTER SYSTEM DECOMMISSION backend "be_host:be_heartbeat_service_port";
```

* DROP 方法で BE クラスタをスケールインする。

> 警告：DROP 方法で BE ノードを削除する場合は、システムの三重レプリカが完全であることを確認してください。

```sql
ALTER SYSTEM DROP backend "be_host:be_heartbeat_service_port";
```

完了したら、ノード情報を確認してスケールインが成功したかどうかを検証できます。

```sql
SHOW PROC '/backends';
```

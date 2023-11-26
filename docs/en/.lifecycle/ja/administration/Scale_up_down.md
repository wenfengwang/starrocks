---
displayed_sidebar: "Japanese"
---

# スケールインとスケールアウト

このトピックでは、StarRocksのノードのスケールインとスケールアウトの方法について説明します。

## FEのスケールインとスケールアウト

StarRocksには、FollowerとObserverの2種類のFEノードがあります。Followerは選挙投票と書き込みに関与します。Observerはログの同期と読み取り性能の拡張にのみ使用されます。

> * フォロワーFEの数（リーダーを含む）は奇数でなければならず、HA（高可用性）モードを形成するために3つのFEを展開することが推奨されます。
> * FEが高可用性展開（1つのリーダー、2つのフォロワー）の場合、読み取り性能を向上させるためにObserver FEを追加することを推奨します。 * 通常、1つのFEノードは10〜20のBEノードと連携することができます。FEノードの総数は10以下であることが推奨されます。ほとんどの場合、3つで十分です。

### FEのスケールアウト

FEノードを展開し、サービスを開始した後、次のコマンドを実行してFEをスケールアウトします。

~~~sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
~~~

### FEのスケールイン

FEのスケールインはスケールアウトと同様です。次のコマンドを実行してFEをスケールインします。

~~~sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
~~~

拡張と収縮後、`show proc '/frontends';`を実行してノード情報を表示できます。

## BEのスケールインとスケールアウト

BEがスケールインまたはスケールアウトされた後、StarRocksは自動的に負荷分散を行い、全体的なパフォーマンスに影響を与えません。

### BEのスケールアウト

次のコマンドを実行してBEをスケールアウトします。

~~~sql
alter system add backend 'be_host:be_heartbeat_service_port';
~~~

BEのステータスを確認するには、次のコマンドを実行します。

~~~sql
show proc '/backends';
~~~

### BEのスケールイン

BEノードのスケールインには2つの方法があります - `DROP`と`DECOMMISSION`。

`DROP`はBEノードをすぐに削除し、失われた重複はFEのスケジューリングによって補完されます。`DECOMMISSION`は、まず重複が補完され、その後BEノードが削除されることを保証します。`DECOMMISSION`は少しフレンドリーであり、BEのスケールインには推奨されます。

両方の方法のコマンドは似ています：

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

バックエンドの削除は危険な操作ですので、実行する前に2回確認する必要があります。

* `alter system drop backend "be_host:be_heartbeat_service_port";`

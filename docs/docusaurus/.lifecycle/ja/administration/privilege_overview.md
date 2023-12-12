---
displayed_sidebar: "Japanese"
---

# 特権の概要

このトピックでは、StarRocksの特権システムの基本的なコンセプトについて説明します。特権は、ユーザーがどのオブジェクトに対してどの操作を実行できるかを決定するものであり、細かく管理することでデータとリソースをより安全に管理することができます。

> 注意: このトピックで説明する特権は、v3.0以降でのみ利用可能です。v3.0の特権フレームワークと構文は、以前のバージョンと互換性がありません。v3.0へのアップグレード後、元の特権のほとんどは保持されますが、特定の操作に関する特権は除外されます。詳細な違いについては、「[StarRocksでサポートされる特権](privilege_item.md)」の「[アップグレードの注意事項](privilege_item_zh.md#%E7%89%B9%E6%A8%A9%E6%94%AF%E6%8C%81%E7%89%88%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)」で確認してください。

StarRocksでは、以下の2つの特権モデルを採用しています。

- ロールベースのアクセス制御（RBAC）：特権はロールに割り当てられ、それがユーザーに割り当てられます。この場合、特権はロールを介してユーザーに伝えられます。
- アイデンティティベースのアクセス制御（IBAC）：特権は直接ユーザーのアイデンティティに割り当てられます。

したがって、各ユーザーアイデンティティの最大特権範囲は、そのユーザーアイデンティティ自体の特権とこのユーザーアイデンティティに割り当てられたロールの特権の合計です。

StarRocksの特権システムを理解するための**基本的なコンセプト**:

- **オブジェクト**: アクセスを許可できるエンティティです。許可されていない場合、アクセスは拒否されます。オブジェクトの例には、CATALOG、DATABASE、TABLE、VIEWなどがあります。詳細については、「[StarRocksでサポートされる特権](privilege_item.md)」を参照してください。
- **特権**: オブジェクトに対するアクセスレベルを定義します。オブジェクトごとに特権が異なる場合があります。特権はオブジェクト固有のものです。特定のオブジェクトに対してどのような操作が許可されているかを定義します。特権の例にはSELECT、ALTER、DROPなどがあります。
- **ユーザーアイデンティティ**: 特権を割り当てることができるユニークなアイデンティティです。ユーザーアイデンティティは`username@'userhost'`という形式で表され、ユーザー名とユーザーがログインするIPから構成されます。アイデンティティの使い方は属性の設定を簡略化します。同じユーザー名を共有するユーザーアイデンティティは同じ属性を共有します。ユーザー名に属性を設定すると、その属性はこのユーザー名を共有するすべてのユーザーアイデンティティに影響します。
- **ロール**: 特権を割り当てることができるエンティティです。ロールは特権の抽象的なコレクションです。ロールはユーザーに割り当てられることもあります。ロールは他のロールにも割り当てることができ、ロールの階層を作成することでデータ管理を容易にします。StarRocksでは、システムで定義されたロールを提供しています。より柔軟性を持たせるために、ビジネスの要件に応じてカスタムロールを作成することもできます。

以下の図は、RBACとIBACの特権モデルの下での特権管理の例を示しています。

これらのモデルでは、特権はロールやユーザーに割り当てられることでオブジェクトへのアクセスが許可されます。ロールは他のロールやユーザーに割り当てられることもあります。

![特権管理](../assets/privilege-manage.png)

## オブジェクトと特権

オブジェクトには論理的な階層があり、それらが表すコンセプトと関連しています。たとえば、データベースはカタログに含まれ、テーブル、ビュー、マテリアライズドビュー、および関数はデータベースに含まれます。以下の図は、StarRocksシステム内のオブジェクトの階層を示しています。

![特権オブジェクト](../assets/privilege-object.png)

それぞれのオブジェクトには、付与できる一連の特権アイテムがあります。これらの特権は、これらのオブジェクトに対して実行できる操作を定義します。[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)コマンドと[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)コマンドを使用して、特権を役割またはユーザーから付与および取り消すことができます。

## ユーザー

### ユーザーアイデンティティ

StarRocksでは、ユーザーは固有のユーザーIDで識別されます。ユーザーIDは、IPアドレス（ユーザーホスト）とユーザー名の形式で構成された`username @'userhost'`です。StarRocksは、異なるIPアドレスからの同じユーザー名のユーザーを異なるユーザーアイデンティティとして識別します。たとえば、`user1@'starrocks.com'`と`user1@'mirrorship.com'`は2つのユーザーアイデンティティです。

ユーザーアイデンティティの別の表現方法は、`username @['domain']`です。ここで、`domain`はDNSで解決できる一連のIPアドレスとして解釈されるドメイン名です。`username @['domain']`は最終的には`username@'userhost'`の集合として表されます。`userhost`の部分には模糊マッチのために`%`を使用できます。`userhost`が指定されていない場合は、デフォルトで`'%'`になります。つまり、同じ名前のユーザーが任意のホストからログインした場合でも、同じ名前を持つユーザーアイデンティティがあります。

### ユーザーへの特権の付与

特権は特権を付与できるエンティティです。特権とロールの両方をユーザーに割り当てることができます。各ユーザーアイデンティティの最大特権範囲は、そのユーザーアイデンティティ自体の特権とこのユーザーアイデンティティに割り当てられたロールの特権の合計です。StarRocksは、各ユーザーが許可された操作のみを実行できるようにします。

ほとんどの場合、**特権を伝達するためにロールを使用**することをお勧めします。例えば、ロールを作成した後、そのロールに特権を付与し、そのロールをユーザーに割り当てることができます。一時的または特別な特権を付与する場合は、特権を直接ユーザーに付与することができます。これにより、特権の管理が簡素化され、柔軟性が提供されます。

## ロール

ロールは特権を付与および取り消すことができるエンティティです。ロールは、必要なアクションを実行するためにユーザーに割り当てられる特権のコレクションと見なすことができます。ユーザーは複数のロールを割り当てることができ、異なる特権のセットを使用して異なるアクションを実行することができます。管理を簡素化するために、StarRocksでは**ロールを介して特権を管理する**ことを推奨します。特別な特権や一時的な特権は、ユーザーに直接付与することができます。

管理を容易にするために、StarRocksはいくつかの**システムで定義されたロール**を提供しています。これにより、日常の管理およびメンテナンスの要件を満たすことができます。また、特定のビジネス要件やセキュリティ要件に合わせて、柔軟に**カスタムロール**を作成することもできます。なお、システムで定義されたロールの特権範囲は変更できません。

ロールがアクティブになると、そのロールで許可された操作をユーザーが実行できるようになります。ユーザーがログインするときに自動的にアクティブになる**デフォルトのロール**を設定することができます。ユーザーは現在のセッションで所有するロールを手動でアクティブにすることもできます。

### システムで定義されたロール

StarRocksでは、いくつかのタイプのシステムで定義されたロールを提供しています。

![ロール](../assets/privilege-role.png)

- `root`: グローバル特権を持っています。デフォルトでは、`root`ユーザーには`root`ロールがあります。
   StarRocksクラスタが作成された後、システムは自動的にrootユーザーを生成し、root特権を持たせます。ルートユーザーとロールにはシステムのすべての特権があるため、リスクのある操作を避けるために、後続の操作には新しいユーザーとロールを作成することをおすすめします。rootユーザーのパスワードを適切に保管してください。
- `cluster_admin`: ノード関連の操作（ノードの追加や削除など）を実行するためのクラスタ管理特権を持っています。
  `cluster_admin`は、クラスタノードの追加、削除、退役を行う特権を持っています。`cluster_admin`またはこのロールをデフォルトのロールとして含む任意のカスタムロールをユーザーに割り当てないようにすることで、予期しないノードの変更を防ぐことをお勧めします。
- `db_admin`: カタログ、データベース、テーブル、ビュー、マテリアライズドビュー、関数、グローバル関数、リソースグループ、プラグインなどに対するすべての操作を行うためのデータベース管理特権を持っています。
- `user_admin`: ユーザーとロールに対する管理特権を持っています。ユーザー、ロール、特権を作成する特権を持っています。

  上記のシステムで定義されたロールは、複雑なデータベースの特権をまとめるために設計されています。**上記のロールの特権範囲は変更できません。**

  また、特定の特権をすべてのユーザーに付与する必要がある場合、StarRocksはシステムで定義されたロール`public`も提供しています。

- `public`: このロールは任意のユーザーに所属し、セッション内でデフォルトでアクティブ化されます。`public`ロールはデフォルトでは特権を持ちません。このロールの特権範囲を変更することができます。

### カスタムロール

特定のビジネス要件を満たすためにカスタムロールを作成し、その特権範囲を変更することができます。また、管理のためにロールを他のロールに割り当てて特権の階層と継承を作成することができます。その後、ロールに関連付けられた特権は他のロールで継承されます。

#### ロールの階層と特権の継承

以下の図は、特権の継承の例を示しています。

> 注意: ロールの最大継承レベルは16です。継承関係は双方向にはなりません。

![ロールの継承](../assets/privilege-role_inheri.png)

図に示すように、次の関係があります。

- `role_s`は`role_p`に割り当てられています。`role_p`は`role_s`の`priv_1`を暗黙的に継承します。
- `role_p`は`role_g`に割り当てられ、`role_g`は`role_p`の`priv_2`と`role_s`の`priv_1`を暗黙的に継承します。
- ロールがユーザーに割り当てられた後、ユーザーもこのロールの特権を持ちます。

### アクティブロール

アクティブロールは、ユーザーが現在のセッションでロールの特権を適用できるようにする機能です。`SELECT CURRENT_ROLE();`を使用して、現在のセッションでのアクティブロールを表示できます。詳細については、「[current_role](../sql-reference/utility-functions/current_role.md)」を参照してください。

#### デフォルトのロール

デフォルトのロールは、ユーザーがクラスタにログインする際に自動的にアクティブになるロールです。1人以上のユーザーが所有するロールである場合があります。管理者は、[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)でデフォルトのロールを設定し、[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)でデフォルトのロールを変更できます。

ユーザーは[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)を使用してデフォルトのロールを変更することもできます。
```
Default roles provide basic privilege protection for users. For example, User A has `role_query` and `role_delete`, which has query and delete privilege respectively. We recommend that you only use `role_query` as the default role to prevent data loss caused by high-risk operations such as `DELETE` or `TRUNCATE`. If you need to perform these operations, you can do it after manually setting active roles.

A user who does not have a default role still has the `public` role, which is automatically activated after the user logs in to the cluster.

#### Manually activate roles

In addition to default roles, users can also manually activate one or more existing roles within a session. You can use [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) to view the privileges and roles that can be activated, and use [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) to configure active roles that are effective in the current session.

Note that the SET ROLE command overwrites each other. For example, after a user logs in, the `default_role` is activated by default. Then the user runs `SET ROLE role_s`. At this time, the user has only the privileges of `role_s` and their own privileges. `default_role` is overwritten.

## References

- [Privileges supported by StarRocks](privilege_item.md)
- [Manage user privileges](User_privilege.md)
```
---
displayed_sidebar: Chinese
---
---
displayed_sidebar: "Chinese"
---

# ユーザー権限の管理

import UserPrivilegeCase from '../assets/commonMarkdown/userPrivilegeCase.md'

この記事では、StarRocksでユーザー、ロール、および権限を管理する方法について説明します。

StarRocksは、ロールベースのアクセス制御（RBAC）とアイデンティティベースのアクセス制御（IBAC）の両方を採用してクラスター内の権限を管理し、クラスター管理者が異なる粒度レベルでクラスター内の権限を簡単に制限できるようにしています。

StarRocksクラスターでは、ユーザーやロールに権限を付与できます。ロールは一連の権限であり、必要に応じてクラスター内のユーザーやロールに付与できます。ユーザーやロールは一つ以上のロールを付与されることがあり、これらのロールが異なるオブジェクトに対する権限を決定します。

## ユーザーとロール情報の表示

`user_admin`というシステムにプリセットされたロールを持つユーザーは、StarRocksクラスター内のユーザーとロールの情報を表示できます。

### 権限情報の表示

[SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)を使用して、ユーザーやロールに付与された権限を表示できます。

- 現在のユーザーの権限を表示します。

  ```SQL
  SHOW GRANTS;
  ```

  > **注釈**
  >
  > どのユーザーでも自分の権限を表示できます。特別な権限は必要ありません。

- 特定のユーザーの権限を表示します。

  以下の例では、ユーザー`jack`の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 特定のロールの権限を表示します。

  以下の例では、ロール`example_role`の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### ユーザー属性の表示

[SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md)を使用して、ユーザーの属性を表示できます。

以下の例では、ユーザー`jack`の属性を表示しています：

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### ロールの表示

[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)を使用して、StarRocksクラスター内のすべてのロールを表示できます。

```SQL
SHOW ROLES;
```

### ユーザーの表示

SHOW USERSを使用して、StarRocksクラスター内のすべてのユーザーを表示できます。

```SQL
SHOW USERS;
```

## ユーザーの管理

`user_admin`というシステムにプリセットされたロールを持つユーザーは、StarRocksでユーザーを作成、変更、削除できます。

### ユーザーの作成

ユーザーID（User Identity）、認証方法、およびデフォルトロールを指定してユーザーを作成できます。

StarRocksは、ユーザーパスワードでのログインまたはLDAP認証をユーザー認証方法としてサポートしています。StarRocksの認証方法の詳細については、[ユーザー認証](../administration/Authentication.md)を参照してください。ユーザーの作成に関する詳細な操作説明については、[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)を参照してください。

以下の例では、ユーザー`jack`を作成し、IPアドレス`172.10.1.10`からの接続のみを許可し、パスワードを`12345`に設定し、デフォルトロールとして`example_role`を割り当てています：

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注釈**
>
> - StarRocksは、ユーザーパスワードを保存する前にそれを暗号化します。password()関数を使用して暗号化されたパスワードを取得できます。
> - ユーザーを作成する際にデフォルトロールが指定されていない場合、StarRocksはシステムにプリセットされた`PUBLIC`ロールをユーザーのデフォルトロールとして指定します。

### ユーザーの変更

ユーザーのパスワード、デフォルトロール、または属性を変更できます。

ユーザーがStarRocksに接続すると、そのデフォルトロールが自動的にアクティブになります。接続後にユーザーのすべてのロール（デフォルトと付与されたロール）を有効にする方法については、[すべてのロールを有効にする](#すべてのロールを有効にする)を参照してください。

#### ユーザーのデフォルトロールの変更

[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)または[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)を使用して、ユーザーのデフォルトロールを設定できます。

以下の2つの例はどちらも、`jack`のデフォルトロールを`db1_admin`に設定しています。設定する前に、`db1_admin`ロールが`jack`に付与されていることを確認する必要があります。

- SET DEFAULT ROLEを使用してデフォルトロールを設定：

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- ALTER USERを使用してデフォルトロールを設定：

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### ユーザー属性の変更

[SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md)を使用して、ユーザーの属性を設定できます。

同じユーザー名を持つユーザーIDは、1つの属性を共有します。以下の例では、`jack`に属性を設定するだけで、その属性設定がユーザー名`jack`を持つすべてのユーザーIDに適用されます。

ユーザー`jack`の最大接続数を`1000`に設定します：

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### ユーザーパスワードのリセット

[SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md)または[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)を使用して、ユーザーのパスワードをリセットできます。

> **注釈**
>
> - どのユーザーでも自分のパスワードをリセットできます。特別な権限は必要ありません。
> - `root`ユーザーのパスワードは`root`ユーザー自身のみがリセットできます。パスワードを失念してStarRocksに接続できない場合は、[失われたrootパスワードのリセット](#失われたrootパスワードのリセット)を参照してください。

以下の2つの例はどちらも、`jack`のパスワードを`54321`にリセットしています：

- SET PASSWORDを使用してパスワードをリセット：

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- ALTER USERを使用してパスワードをリセット：

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### 失われたrootパスワードのリセット

rootユーザーのパスワードを失念してStarRocksに接続できない場合は、以下の手順でパスワードをリセットできます：

1. **すべてのFEノード**の設定ファイル**fe/conf/fe.conf**に以下の設定項目を追加して、ユーザー認証を無効にします：

   ```YAML
   enable_auth_check = false
   ```

2. 設定を有効にするために**すべてのFEノード**を再起動します。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. `root`ユーザーでMySQLクライアントからStarRocksに接続します。ユーザー認証が無効の場合、パスワードは不要です。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. `root`ユーザーのパスワードをリセットします。

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. **すべてのFEノード**の設定ファイル**fe/conf/fe.conf**で、設定項目`enable_auth_check`を`true`に設定して、ユーザー認証を再度有効にします。

   ```YAML
   enable_auth_check = true
   ```

6. 設定を有効にするために**すべてのFEノード**を再起動します。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. `root`ユーザーと新しいパスワードでMySQLクライアントからStarRocksに接続して、パスワードが正常にリセットされたかを確認します。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### ユーザーの削除

[DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md)を使用して、ユーザーを削除できます。

以下の例では、ユーザー`jack`を削除しています：

```SQL
DROP USER jack@'172.10.1.10';
```

## ロールの管理

`user_admin`というシステムにプリセットされたロールを持つユーザーは、StarRocksでロールを作成、付与、取り消し、削除できます。

### ロールの作成

[CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md)を使用して、ロールを作成できます。

以下の例では、ロール`example_role`を作成しています：

```SQL
CREATE ROLE example_role;
```

### ロールの付与

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を使用して、ロールをユーザーや他のロールに付与できます。

- ユーザーにロールを付与する。

  以下の例では、ロール `example_role` をユーザー `jack` に付与しています:

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- 他のロールにロールを付与する。

  以下の例では、ロール `example_role` をロール `test_role` に付与しています:

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### ロールの剥奪

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーや他のロールからロールを剥奪することができます。

> **説明**
>
> システムにプリセットされたデフォルトロール `PUBLIC` は剥奪できません。

- ユーザーからロールを剥奪する。

  以下の例では、ユーザー `jack` からロール `example_role` を剥奪しています:

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- ロールから他のロールを剥奪する。

  以下の例では、ロール `test_role` からロール `example_role` を剥奪しています:

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### ロールの削除

[DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md) を使用して、ロールを削除することができます。

以下の例では、ロール `example_role` を削除しています:

```SQL
DROP ROLE example_role;
```

> **注意**
>
> システムにプリセットされたロールは削除できません。

### すべてのロールを有効にする

ユーザーのデフォルトロールは、ユーザーが StarRocks クラスタに接続するたびに自動的にアクティブになるロールです。ロールに付与された権限は、付与された後にのみ有効になります。

ログイン時にクラスタ内のすべてのユーザーにデフォルトですべてのロール（デフォルトと付与されたロール）をアクティブにしたい場合は、以下の操作を実行できます。この操作には system レベルの OPERATE 権限が必要です。

以下のステートメントを実行して、クラスタ内のすべてのユーザーにすべてのロールを有効にします：

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

また、SET ROLE を使用して手動で所有するロールをアクティブにすることもできます。例えば、ユーザー jack@'172.10.1.10' は `db_admin` と `user_admin` のロールを持っていますが、これらは彼のデフォルトロールではありませんので、ログイン時にはデフォルトでアクティブになりません。jack@'172.10.1.10' が `db_admin` と `user_admin` をアクティブにする必要がある場合は、手動で `SET ROLE db_admin, user_admin;` を実行できます。SET ROLE コマンドは上書きされるので、所有するすべてのロールをアクティブにしたい場合は、SET ROLE ALL を実行できます。

## 権限管理

システムにプリセットされた `user_admin` ロールを持つユーザーは、StarRocks で権限を付与および剥奪することができます。

### 権限の付与

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を使用して、ユーザーやロールに権限を付与することができます。

- ユーザーに権限を付与する。

  以下の例では、テーブル `sr_member` の SELECT 権限をユーザー `jack` に付与し、`jack` がこの権限を他のユーザーやロールに付与できるようにしています（SQL で WITH GRANT OPTION を指定することにより）:

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- ロールに権限を付与する。

  以下の例では、テーブル `sr_member` の SELECT 権限をロール `example_role` に付与しています:

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 権限の剥奪

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーやロールの権限を剥奪することができます。

- ユーザーの権限を剥奪する。

  以下の例では、ユーザー `jack` からテーブル `sr_member` の SELECT 権限を剥奪し、`jack` がこの権限を他のユーザーやロールに付与することを禁止しています:

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- ロールの権限を剥奪する。

  以下の例では、ロール `example_role` からテーブル `sr_member` の SELECT 権限を剥奪しています:

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## ベストプラクティス

### 複数ビジネスラインの権限管理

通常、企業内部では、StarRocks クラスタはプラットフォーム側によって一元管理され、各種ビジネス側にサービスを提供しています。その中で、一つの StarRocks クラスタ内には複数のビジネスラインが含まれており、それぞれのビジネスラインは一つ以上のデータベースに関連している可能性があります。

例えば、組織構造にはプラットフォーム側とビジネス側が含まれています。ビジネス側にはビジネスライン A と B があり、それぞれにはアナリストやセールスなどの異なる役割があります。アナリストは日常的にレポートや分析レポートを作成する必要があり、セールスはアナリストが作成したレポートを日常的に照会する必要があります。

![ユーザー権限](../assets/user_privilege_1.png)

データ構造では、ビジネス A と B はそれぞれ自分のデータベース `DB_A` と `DB_B` を持っています。データベース `DB_C` では、ビジネス A と B はいくつかのテーブルを共有しています。また、会社の全員が公共データベース `DB_PUBLIC` にアクセスできます。

![ユーザー権限](../assets/user_privilege_2.png)

異なるビジネスや役割には日常業務や関連するデータベースやテーブルが異なるため、StarRocks はビジネスや役割に応じてロールを作成し、必要な権限を対応するロールに付与した後、ユーザーに割り当てることを推奨しています。具体的には：

![ユーザー権限](../assets/user_privilege_3.png)

1. システムにプリセットされたロール `db_admin`、`user_admin`、および `cluster_admin` をプラットフォーム運用ロールに付与します。同時に、`db_admin` と `user_admin` をデフォルトロールとして設定し、日常の基本的な運用に使用します。ノード操作が必要と確認された場合は、`cluster_admin` ロールを手動でアクティブにします。

   例えば：

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. プラットフォーム運用スタッフがシステム内のすべてのユーザーを作成し、それぞれに複雑なパスワードを設定します。
3. 各ビジネス側に機能に応じたロールを設定し、例えばこの例ではビジネス管理者、ビジネスアナリスト、セールスなどのロールを設定し、それぞれに対応する権限を付与します。

   ビジネス責任者には、そのビジネスに必要な権限の最大セットを付与し、さらに権限付与権限（つまり、GRANT 時に WITH GRANT OPTION キーワードを追加する）を付与します。これにより、後続の作業で自分の部下に必要な権限を割り当てることができます。このロールが彼らの日常的なロールであれば、デフォルトロールとして設定することができます。

   例えば：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```

   アナリストやセールスなどのロールには、それぞれの操作権限を付与します。

   例えば：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. 誰でもアクセスできる公共データベースについては、そのデータベースのすべてのテーブルのクエリ権限をプリセットロール `public` に付与します。

  例えば：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

複雑な状況では、ロールを他のロールに付与することで、権限の継承を実現することもできます。

複雑な状況下では、他のロールにロールを割り当てることで、権限の継承を実現することもできます。

例えば、すべてのアナリストがDB_PUBLICのデータに対してインポートや変更を行うことができ、すべての営業担当者がDB_PUBLICのデータに対してクエリを実行することができます。public_analysisとpublic_salesを作成し、対応する権限を付与した後、それらのロールをすべてのビジネスラインのアナリストと営業担当者のロールに割り当てることができます。

```SQL
CREATE ROLE public_analysis;
CREATE ROLE public_sales;
GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public_analysis;
GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public_sales;
GRANT public_analysis TO ROLE linea_analysis;
GRANT public_analysis TO ROLE lineb_analysis;
GRANT public_sales TO ROLE linea_query;
GRANT public_sales TO ROLE lineb_query;
```

### 使用シナリオに基づいてカスタムロールを作成する

<UserPrivilegeCase />

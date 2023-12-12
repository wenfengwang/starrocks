---
displayed_sidebar: "Japanese"
---

# ユーザ権限を管理する

import UserPrivilegeCase from '../assets/commonMarkdown/userPrivilegeCase.md'

このトピックでは、StarRocksでユーザ、ロール、権限の管理方法について説明します。

StarRocksは、StarRocksクラスタ内で権限を簡単に制限するために、ロールベースのアクセス制御（RBAC）およびアイデンティティベースのアクセス制御（IBAC）の両方を採用しています。

StarRocksクラスタ内では、権限をユーザまたはロールに付与することができます。ロールは、必要に応じてクラスタ内の他のユーザやロールに割り当てることができる権限のコレクションです。ユーザは1つまたは複数のロールを付与されることができ、それによってさまざまなオブジェクトに対する権限が決まります。

## ユーザとロール情報を表示する

システム定義ロール `user_admin` を持つユーザは、StarRocksクラスタ内のすべてのユーザとロール情報を表示することができます。

### 権限情報を表示する

[SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)を使用して、ユーザまたはロールに付与された権限を表示することができます。

- 現在のユーザの権限を表示する。

  ```SQL
  SHOW GRANTS;
  ```

  > **注意**
  >
  > 任意のユーザは、権限が不要で自分自身の権限を表示することができます。

- 特定のユーザの権限を表示する。

  次の例は、ユーザ `jack` の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 特定のロールの権限を表示する。

  次の例は、ロール `example_role` の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### ユーザプロパティを表示する

[SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md)を使用して、ユーザのプロパティを表示することができます。

次の例は、ユーザ `jack` のプロパティを表示しています。

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### ロールを表示する

[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)を使用して、StarRocksクラスタ内のすべてのロールを表示することができます。

```SQL
SHOW ROLES;
```

### ユーザを表示する

SHOW USERS を使用して、StarRocksクラスタ内のすべてのユーザを表示することができます。

```SQL
SHOW USERS;
```

## ユーザを管理する

システム定義ロール `user_admin` を持つユーザは、StarRocksでユーザを作成、変更、削除することができます。

### ユーザを作成する

ユーザの識別子、認証方法、およびデフォルトのロールを指定してユーザを作成することができます。

StarRocksでは、ログイン資格情報またはLDAP認証を使用したユーザ認証をサポートしています。StarRocksの認証についての詳細については、[認証](../administration/Authentication.md)を参照してください。ユーザの作成に関する詳細な手順については、[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)を参照してください。

次の例では、ユーザ `jack` を作成し、IPアドレス `172.10.1.10` からのみ接続を許可し、そのパスワードを `12345` に設定し、デフォルトのロールとして `example_role` を割り当てています。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注意**
>
> - StarRocksは、ユーザのパスワードを保存する前に暗号化します。暗号化されたパスワードは、password() 関数を使用して取得できます。
> - ユーザの作成時にデフォルトのロールが指定されていない場合、システム定義のデフォルトロール `PUBLIC` がユーザに割り当てられます。

### ユーザを変更する

ユーザのパスワード、デフォルトのロール、またはプロパティを変更することができます。

ユーザがStarRocksに接続すると、デフォルトロールが自動的に有効化されます。接続後にユーザのすべて（デフォルトと付与された）のロールを有効にする方法については、[すべてのロールを有効にする](#すべてのロールを有効にする)を参照してください。

#### ユーザのデフォルトロールを変更する

[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)または[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)を使用して、ユーザのデフォルトロールを設定することができます。

次の例では、`jack` のデフォルトロールを `db1_admin` に設定しています。なお、`db1_admin` は `jack` に割り当てている必要があります。

- SET DEFAULT ROLE を使用してデフォルトロールを設定する。

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- ALTER USER を使用してデフォルトロールを設定する。

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### ユーザのプロパティを変更する

[SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md)を使用して、ユーザのプロパティを設定することができます。

次の例では、ユーザ `jack` の最大接続数を `1000` に設定しています。同じユーザ名を持つユーザ識別子のプロパティは共有されるため、`jack` のプロパティを設定することで、ユーザ名が `jack` のすべてのユーザ識別子でこの設定が有効になります。

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### ユーザのパスワードをリセットする

[SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md)または[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)を使用して、ユーザのパスワードをリセットすることができます。

> **注意**
>
> - 任意のユーザは、権限が不要で自分自身のパスワードをリセットすることができます。
> - `root` ユーザ自体だけがパスワードを設定できます。パスワードを失った場合でStarRocksに接続できない場合は、詳細な手順については [失われた root パスワードをリセットする](#失われた-root-パスワードをリセットする)を参照してください。

次の例は、`jack` のパスワードを `54321` にリセットしています。

- SET PASSWORD を使用してパスワードをリセットする。

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- ALTER USER を使用してパスワードをリセットする。

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### 失われた root パスワードをリセットする

`root` ユーザのパスワードを失った場合でStarRocksに接続できない場合は、次の手順に従ってリセットすることができます。

1. **すべてのFEノード**の設定ファイル **fe/conf/fe.conf** に、ユーザ認証を無効にするための次の構成項目を追加します。

   ```YAML
   enable_auth_check = false
   ```

2. **すべてのFEノード**を再起動して、構成が有効になるようにします。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. MySQLクライアントから、ユーザ認証が無効になっているためパスワードを指定する必要がない `root` ユーザを使用してStarRocksに接続します。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. `root` ユーザのパスワードをリセットします。

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. **すべてのFEノード**の設定ファイル **fe/conf/fe.conf** に、構成項目 `enable_auth_check` を `true` に設定して、ユーザ認証を再度有効にします。

   ```YAML
   enable_auth_check = true
   ```

6. **すべてのFEノード**を再起動して、構成が有効になるようにします。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. `root` ユーザと新しいパスワードを使用してStarRocksに接続し、パスワードが正常にリセットされたかを確認します。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### ユーザを削除する

[DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md)を使用してユーザを削除することができます。

次の例は、ユーザ `jack` を削除しています。

```SQL
DROP USER jack@'172.10.1.10';
```

## ロールを管理する

システム定義ロール `user_admin` を持つユーザは、StarRocksでロールを作成、付与、取り消し、または削除することができます。

### ロールを作成する

[CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md)を使用して、ロールを作成することができます。

次の例は、ロール `example_role` を作成しています。

```SQL
CREATE ROLE example_role;
```

### ロールを付与する

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を使用して、ユーザまたは他のロールにロールを付与することができます。

- ユーザにロールを付与する。

  次の例では、ユーザ `jack` にロール `example_role` を付与しています。

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- 他のロールにロールを付与する。

  次の例は、ロール `example_role` をロール `test_role` に付与しています。

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### ロールを取り消す

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザまたは他のロールからロールを取り消すことができます。

> **注意**
>
> システム定義のデフォルトロール `PUBLIC` をユーザーから取り消すことはできません。

- ユーザーからロールを取り消す

  次の例では、ユーザー `jack` からロール `example_role` を取り消します:

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- 別のロールからロールを取り消す

  次の例では、ロール `test_role` からロール `example_role` を取り消します:

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### ロールの削除

[DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md) を使用してロールを削除できます。

次の例では、ロール `example_role` を削除します:

```SQL
DROP ROLE example_role;
```

> **注意**
>
> システム定義のロールは削除できません。

### すべてのロールを有効にする

ユーザーのデフォルトロールは、ユーザーが StarRocks クラスタに接続する度に自動的にアクティブ化されるロールです。

StarRocks ユーザーが StarRocks クラスタに接続する際に、すべてのロール（デフォルトおよび付与されたロール）を有効にしたい場合は、次の操作を実行できます。

この操作には、システム権限 OPERATE が必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

また、与えられたロールをアクティブ化するには SET ROLE を使用できます。たとえば、ユーザー `jack@'172.10.1.10'` は `db_admin` および `user_admin` のロールを持っていますが、これらはユーザーのデフォルトロールではなく、StarRocks に接続した際に自動的にアクティブ化されるわけではありません。`jack@'172.10.1.10'` が `db_admin` および `user_admin` をアクティブ化する必要がある場合は、`SET ROLE db_admin, user_admin;` を実行します。SET ROLE は元のロールを上書きします。すべてのロールを有効にしたい場合は、SET ROLE ALL を実行します。

## 権限の管理

システム定義のロール `user_admin` を持つユーザーは、StarRocks で権限を付与または取り消すことができます。

### 権限を付与する

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を使用して、ユーザーやロールに権限を付与できます。

- ユーザーに権限を付与する

  次の例では、テーブル `sr_member` に対する SELECT 権限をユーザー `jack` に付与し、`jack` が他のユーザーやロールにこの権限を付与できるようにします（SQL で WITH GRANT OPTION を指定）:

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- ロールに権限を付与する

  次の例では、テーブル `sr_member` に対する SELECT 権限をロール `example_role` に付与します:

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 権限を取り消す

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーやロールから権限を取り消すことができます。

- ユーザーから権限を取り消す

  次の例では、テーブル `sr_member` に対する SELECT 権限をユーザー `jack` から取り消し、`jack` にこの権限を他のユーザーやロールに付与することを許可しません):

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- ロールから権限を取り消す

  次の例では、テーブル `sr_member` に対する SELECT 権限をロール `example_role` から取り消します:

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## ベストプラクティス

### マルチサービス アクセス制御

通常、企業所有の StarRocks クラスタは、単一のサービスプロバイダーによって管理され、それぞれが1つ以上のデータベースを使用する複数の事業部門ライン（LOB）を維持します。

以下に示すように、StarRocks クラスタのユーザーには、サービスプロバイダーとLOB（A および B）のメンバーが含まれます。各LOBはアナリストとエグゼクティブによって運営されます。アナリストはビジネスレポートを生成し分析し、エグゼクティブはレポートをクエリします。

![ユーザー権限](../assets/user_privilege_1.png)

LOB A はデータベース `DB_A` を独自に管理し、LOB B はデータベース `DB_B` を管理します。LOB A とLOB B は `DB_C` の異なるテーブルを使用します。`DB_PUBLIC` は両LOBのすべてのメンバーがアクセスできます。

![ユーザー権限](../assets/user_privilege_2.png)

異なるメンバーが異なるデータベースやテーブルで異なる操作を実行するため、各サービスおよびポジションに応じてロールを作成し、各ロールに必要な権限のみを適用し、対応するメンバーにこれらのロールを割り当てることをお勧めします。以下に示します:

1. クラスタのメンテナには、システム定義のロール `db_admin`、`user_admin`、`cluster_admin` を割り当て、日常のメンテナンスにはデフォルトロールとして `db_admin` と `user_admin` を設定し、クラスタのノードを操作する必要があるときには手動でロール `cluster_admin` をアクティブ化します。

   例:

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. 各LOB内のメンバーごとにユーザーを作成し、それぞれのユーザーに複雑なパスワードを設定します。
3. 各LOB内のポジションごとにロールを作成し、各ロールに対応する権限を適用します。

   各LOBのディレクターに対して、LOBが必要とする最大の権限のコレクションとそれに対応する GRANT 権限（ステートメントで WITH GRANT OPTION を指定）を付与します。それにより、LOBのメンバーにこれらの権限を割り当てることができます。日常業務に必要な場合は、ロールをデフォルトロールとして設定します。

   例:

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```

   アナリストおよびエグゼクティブには、対応する権限を持つロールを割り当てます。

   例:

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. すべてのクラスタユーザーがアクセスできるデータベース `DB_PUBLIC` には、システム定義のロール `public` にデータベース `DB_PUBLIC` の SELECT 権限を付与します。

   例:

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

複雑なシナリオでのロールの継承を実現するために、他の人にロールを割り当てることができます。

例えば、アナリストが `DB_PUBLIC` のテーブルに書き込むための権限が必要であり、エグゼクティブはこれらのテーブルをクエリするだけで十分です。そういった場合、`public_analysis` および `public_sales` というロールを作成し、関連する権限をロールに付与し、それらを元のアナリストおよびエグゼクティブのロールに割り当てます。

例:

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

### シナリオに基づいたロールのカスタマイズ

<UserPrivilegeCase />
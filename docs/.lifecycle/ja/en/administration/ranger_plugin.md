---
displayed_sidebar: English
---

# Apache Ranger でアクセス許可を管理する

[Apache Ranger](https://ranger.apache.org/) は、ユーザーが視覚的なウェブページを介してアクセスポリシーをカスタマイズできる一元化されたセキュリティ管理フレームワークを提供します。これにより、どのロールがどのデータにアクセスできるかを判断し、Hadoop エコシステムのさまざまなコンポーネントやサービスに対してきめ細かなデータアクセス制御を実行できます。

Apache Ranger には、以下のコアモジュールがあります：

- Ranger Admin：Ranger のコアモジュールで、組み込まれたウェブページがあります。ユーザーは、このページまたは REST インターフェースを使用して、セキュリティポリシーを作成および更新できます。Hadoop エコシステムのさまざまなコンポーネントのプラグインは、これらのポリシーを定期的にポーリングし、取得します。
- Agent Plugin：Hadoop エコシステムに組み込まれたコンポーネントのプラグインです。これらのプラグインは、Ranger Admin から定期的にセキュリティポリシーを取得し、ポリシーをローカルファイルに保存します。ユーザーがコンポーネントにアクセスすると、対応するプラグインは設定されたセキュリティポリシーに基づいてリクエストを評価し、認証結果を対応するコンポーネントに送信します。
- User Sync：ユーザーおよびユーザーグループ情報を取得し、ユーザーおよびユーザーグループの権限データを Ranger のデータベースに同期するために使用されます。

ネイティブの RBAC 権限システムに加えて、StarRocks v3.1 は Apache Ranger を通じたアクセス制御もサポートし、より高いレベルのデータセキュリティを提供します。

このトピックでは、StarRocks と Apache Ranger の権限制御方法と統合プロセスについて説明します。Ranger でセキュリティポリシーを作成してデータセキュリティを管理する方法については、[Apache Ranger の公式ウェブサイト](https://ranger.apache.org/)を参照してください。

## 権限制御方法

Apache Ranger と統合された StarRocks は、以下の権限制御方法を提供します：

- Ranger で StarRocks サービスを作成し、権限制御を実装します。ユーザーが StarRocks の内部テーブル、外部テーブル、またはその他のオブジェクトにアクセスする場合、StarRocks サービスで設定されたアクセスポリシーに従ってアクセス制御が実行されます。
- ユーザーが外部データソースにアクセスする場合、Apache Ranger 上の外部サービス（例えば Hive サービス）をアクセス制御に再利用できます。StarRocks は Ranger サービスを異なる External Catalogs と照合し、データソースに対応する Ranger サービスに基づいてアクセス制御を実装します。

StarRocks が Apache Ranger と統合された後、以下のアクセス制御パターンを実現できます：

- Apache Ranger を使用して、StarRocks の内部テーブル、外部テーブル、およびすべてのオブジェクトへのアクセスを一元的に管理します。
- Apache Ranger を使用して、StarRocks の内部テーブルおよびオブジェクトへのアクセスを管理します。External Catalogs の場合、Ranger 上の対応する外部サービスのポリシーをアクセス制御に再利用します。
- Apache Ranger を使用して、外部データソースに対応するサービスを再利用して、External Catalogs へのアクセスを管理します。StarRocks の RBAC 権限システムを使用して、StarRocks の内部テーブルおよびオブジェクトへのアクセスを管理します。

**認証プロセス**

- ユーザー認証に LDAP を使用し、その後 Ranger を使用して LDAP ユーザーを同期し、アクセスルールを構成することもできます。StarRocks は LDAP を介してユーザーログイン認証を完了することもできます。
- ユーザーがクエリを開始すると、StarRocks はクエリステートメントを解析し、ユーザー情報と必要な権限を Apache Ranger に渡します。Ranger は、該当するサービスで設定されたアクセスポリシーに基づいて、ユーザーが必要な権限を持っているかどうかを判断し、認証結果を StarRocks に返します。ユーザーがアクセス権を持っている場合、StarRocks はクエリデータを返します。そうでない場合、StarRocks はエラーを返します。

## 前提条件

- Apache Ranger 2.1.0 以降がインストールされています。Apache Ranger のインストール方法については、[Ranger クイックスタート](https://ranger.apache.org/quick_start_guide.html)を参照してください。
- すべての StarRocks FE マシンは Apache Ranger にアクセスできます。これを確認するには、各 FE マシンで以下のコマンドを実行します：

   ```SQL
   telnet <ranger-ip> <ranger-port>
   ```

   `Connected to <ip>` が表示されれば、接続は成功しています。

## 統合手順

### ranger-starrocks-plugin のインストール

現在、StarRocks は以下をサポートしています：

- Apache Ranger を使用してアクセスポリシー、マスキングポリシー、および行レベルのフィルターポリシーを作成します。
- Ranger 監査ログ。

1. Ranger Admin ディレクトリ `ews/webapp/WEB-INF/classes/ranger-plugins` に `starrocks` フォルダを作成します。

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```


2. [plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar](https://www.starrocks.io/download/community)と[mysql-connector-j](https://dev.mysql.com/downloads/connector/j/)をダウンロードし、`starrocks`フォルダに配置します。

3. Ranger Adminを再起動します。

   ```SQL
   ranger-admin restart
   ```

### Ranger AdminでStarRocksサービスを設定する

1. [ranger-servicedef-starrocks.json](https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json)をStarRocks FEマシンまたはRangerマシンの任意のディレクトリにコピーします。

   ```SQL
   wget https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json
   ```

2. Ranger管理者として以下のコマンドを実行し、StarRocksサービスを追加します。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> \
   -X POST -H "Accept: application/json" \
   -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. `http://<ranger-ip>:<ranger-port>/login.jsp`にアクセスしてApache Rangerページにログインします。STARROCKSサービスがページに表示されます。

   ![home](../assets/ranger_home.png)

4. **STARROCKS**の後のプラス記号(`+`)をクリックしてStarRocksサービスを設定します。

   ![service detail](../assets/ranger_service_details.png)

   ![property](../assets/ranger_properties.png)

   - `Service Name`: サービス名を入力する必要があります。
   - `Display Name`: STARROCKSの下に表示したいサービス名。指定されていない場合は`Service Name`が表示されます。
   - `Username`と`Password`: FEのユーザー名とパスワードは、ポリシー作成時にオブジェクト名を自動補完するために使用されます。これら2つのパラメータはStarRocksとRangerの接続性には影響しません。自動補完を使用する場合は、`db_admin`ロールが有効になっているユーザーを少なくとも1人設定してください。
   - `jdbc.url`: StarRocks FEのIPアドレスとポートを入力します。

   以下の図は設定例を示しています。

   ![example](../assets/ranger_show_config.png)

   以下の図は追加されたサービスを示しています。

   ![added service](../assets/ranger_added_service.png)

5. **Test connection**をクリックして接続をテストし、接続が成功したら保存します。
6. StarRocksクラスタの各FEマシンで、`fe/conf`フォルダに[ranger-starrocks-security.xml](https://github.com/StarRocks/ranger/blob/master/plugin-starrocks/conf/ranger-starrocks-security.xml)を作成し、内容をコピーします。以下の2つのパラメータを変更し、変更を保存してください：

   - `ranger.plugin.starrocks.service.name`: ステップ4で作成したStarRocksサービスの名前に変更します。
   - `ranger.plugin.starrocks.policy.rest.url`: Ranger Adminのアドレスに変更します。

   他の設定を変更する必要がある場合は、Apache Rangerの公式ドキュメントを参照してください。例えば、`ranger.plugin.starrocks.policy.pollIntervalMs`を変更してポリシー変更のプル間隔を変更することができます。

   ```SQL
   vim ranger-starrocks-security.xml

   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- Change it to the StarRocks Service name.
           <description>
               This StarRocks instance's Ranger service containing policies
           </description>
       </property>
   ...

   ...
       <property>
           <name>ranger.plugin.starrocks.policy.rest.url</name>
           <value>http://localhost:6080</value> -- Change it to Ranger Admin address.
           <description>
               URL to Ranger Admin
           </description>
       </property>   
   ...
   ```

7. すべてのFE設定ファイルに`access_control = ranger`の設定を追加します。

   ```SQL
   vim fe.conf
   access_control=ranger 
   ```

8. すべてのFEマシンを再起動します。

   ```SQL
   -- Switch to the FE folder. 
   cd ..

   bin/stop_fe.sh
   bin/start_fe.sh
   ```

## 外部テーブルへのアクセス制御に他のサービスを再利用する

外部カタログについては、アクセス制御のために外部サービス（例えばHiveサービス）を再利用することができます。StarRocksは異なるカタログに対して異なるRanger外部サービスをマッチングすることをサポートしています。ユーザーが外部テーブルにアクセスする際、システムは外部テーブルに対応するRangerサービスのアクセスポリシーに基づいてアクセス制御を実施します。

1. HiveのRanger設定ファイル`ranger-hive-security.xml`をすべてのFEマシンの`fe/conf`フォルダにコピーします。
2. すべてのFEマシンを再起動します。
3. 外部カタログを設定します。

   - 外部カタログを作成する際に、プロパティ`"ranger.plugin.hive.service.name"`を追加します。

      ```SQL
        CREATE EXTERNAL CATALOG hive_catalog_1
        PROPERTIES (
            "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
            "ranger.plugin.hive.service.name" = "hive_catalog_1"
        )
      ```

   - このプロパティは既存の外部カタログにも追加できます。
  
       ```SQL
       ALTER CATALOG hive_catalog_1
       SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
       ```

この操作により、既存のカタログの認証方法がRangerベースの認証に変更されます。

## 次に行うこと

StarRocksサービスを追加した後、そのサービスをクリックしてアクセス制御ポリシーを作成し、異なるユーザーやユーザーグループに異なる権限を割り当てることができます。ユーザーがStarRocksデータにアクセスする際には、これらのポリシーに基づいてアクセス制御が行われます。

---
displayed_sidebar: English
---

# 使用 Apache Ranger 管理权限

[Apache Ranger](https://ranger.apache.org/) 提供了一个集中式安全管理框架，允许用户通过可视化网页自定义访问策略。这有助于确定哪些角色可以访问哪些数据，并对 Hadoop 生态系统中的各种组件和服务进行细粒度的数据访问控制。

Apache Ranger 提供以下核心模块：

- - Ranger Admin：Ranger 的核心模块，内置网页。用户可以在此页面或通过 REST 接口创建和更新安全策略。Hadoop 生态系统中的各个组件插件会定期轮询并拉取这些策略。
- - Agent Plugin：嵌入在 Hadoop 生态系统中的组件插件。这些插件会定期从 Ranger Admin 拉取安全策略，并将策略存储在本地文件中。当用户访问某个组件时，相应的插件会根据配置的安全策略评估请求，并将认证结果发送给相应组件。
- - 用户同步：用于拉取用户和用户组信息，并将用户和用户组的权限数据同步到 Ranger 的数据库中。

除了原生的 RBAC 权限系统外，StarRocks v3.1 还支持通过 Apache Ranger 进行访问控制，从而提供更高级别的数据安全性。

本文介绍 StarRocks 与 Apache Ranger 的权限控制方法及集成流程。有关如何在 Ranger 上创建安全策略以管理数据安全的信息，请参见[Apache Ranger 官方网站](https://ranger.apache.org/)。

## Permission control method

权限控制方法

- StarRocks 集成 Apache Ranger 后提供以下权限控制方法：
- - 在 Ranger 中创建 StarRocks 服务以实施权限控制。当用户访问 StarRocks 的内部表、外部表或其他对象时，将根据在 StarRocks 服务中配置的访问策略执行访问控制。

- 当用户访问外部数据源时，可以复用 Apache Ranger 上的外部服务（例如 Hive 服务）来进行访问控制。StarRocks 可以将 Ranger 服务与不同的 External Catalog 匹配，并基于数据源相应的 Ranger 服务实施访问控制。

- Use Apache Ranger to uniformly manage access to StarRocks internal tables, external tables, and all objects.
- 集成后的访问控制模式：
- - 使用 Apache Ranger 统一管理对 StarRocks 内部表、外部表及所有对象的访问。

**认证过程**

- - 使用 Apache Ranger 管理对 External Catalogs 的访问，通过重用外部数据源相应的服务。使用 StarRocks 的 RBAC 权限系统来管理对 StarRocks 内部表和对象的访问。
- When users initiate a query, StarRocks parses the query statement, passes user information and required privileges to Apache Ranger. Ranger determines whether the user has the required privilege based on the access policy configured in the corresponding Service, and returns the authentication result to StarRocks. If the user has access, StarRocks returns the query data; if not, StarRocks returns an error.

## 认证流程

- 您还可以使用 LDAP 进行用户认证，然后使用 Ranger 同步 LDAP 用户并为他们配置访问规则。StarRocks 也可以通过 LDAP 完成用户登录认证。
- 当用户发起查询时，StarRocks 解析查询语句，将用户信息和所需权限传递给 Apache Ranger。Ranger 根据相应服务中配置的访问策略判断用户是否具有所需权限，并将认证结果返回给 StarRocks。如果用户有访问权限，StarRocks 返回查询数据；否则，StarRocks 返回错误信息。

  ```SQL
  telnet <ranger-ip> <ranger-host>
  ```
  If `Connected to <ip>` is displayed, the connection is successful.

## 先决条件

### - 已安装 Apache Ranger 2.1.0 或更高版本。有关如何安装 Apache Ranger 的说明，请参阅 Ranger 快速入门指南。

- 所有 StarRocks FE 机器都能访问 Apache Ranger。您可以通过在每台 FE 机器上运行以下命令来检查：

-   如果显示“Connected to <ip>”，则表示连接成功。
- Ranger audit logs.

1. 集成流程

   ```SQL
   mkdir {path-to-ranger}/ews/webapp/WEB-INF/classes/ranger-plugins/starrocks
   ```

2. 下载[plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar](https://www.starrocks.io/download/community)和[mysql-connector-j](https://dev.mysql.com/downloads/connector/j/)，并将它们放在`starrocks`文件夹中。

3. 目前，StarRocks 支持：

   ```SQL
   ranger-admin restart
   ```

### - 通过 Apache Ranger 创建访问策略、掩码策略和行级过滤策略。

1. 复制[ranger-servicedef-starrocks.json](https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json)到StarRocks FE机器或Ranger机器的任意目录。

   ```SQL
   wget https://github.com/StarRocks/ranger/blob/master/agents-common/src/main/resources/service-defs/ranger-servicedef-starrocks.json
   ```

2. - 在 Ranger Admin 目录下的 ews/webapp/WEB-INF/classes/ranger-plugins 中创建 starrocks 文件夹。

   ```SQL
   curl -u <ranger_adminuser>:<ranger_adminpwd> \
   -X POST -H "Accept: application/json" \
   -H "Content-Type: application/json" http://<ranger-ip>:<ranger-port>/service/plugins/definitions -d@ranger-servicedef-starrocks.json
   ```

3. - 下载 plugin-starrocks/target/ranger-starrocks-plugin-3.0.0-SNAPSHOT.jar 和 mysql-connector-j，并将它们放入 starrocks 文件夹。

   ![home](../assets/ranger_home.png) - 重启 Ranger Admin。

4. Click the plus sign (`+`) after **STARROKCS** to configure StarRocks Service.

   ![service detail](../assets/ranger_service_details.png)在 Ranger Admin 上配置 StarRocks 服务

   ![property](../assets/ranger_properties.png)

   - - 作为 Ranger 管理员运行以下命令以添加 StarRocks 服务。
   - - 访问 http://<ranger-ip>:<ranger-port>/login.jsp 登录 Apache Ranger 页面。页面上将显示 STARROCKS 服务。
   - - 点击 STARROCKS 后的加号 (+) 配置 StarRocks 服务。
   - - 输入服务名称和显示名称，这是您希望在 STARROCKS 下显示的服务名。如果未指定，默认显示服务名称。

   - 输入 FE 用户名和密码，用于创建策略时自动完成对象名称。这两个参数不影响 StarRocks 与 Ranger 之间的连接。如果要使用自动完成，至少配置一个启用了 db_admin 角色的用户。

   ![example](../assets/ranger_show_config.png) - 输入 jdbc.url，包括 StarRocks FE 的 IP 地址和端口。

   - 测试连接以确保连通性，并在成功后保存设置。

   ![added service](../assets/ranger_added_service.png)

5. 点击**测试连接**以测试连通性，在连接成功后保存。
6. 在StarRocks集群的每台FE机器上，在`fe/conf`文件夹中创建[ranger-starrocks-security.xml](https://github.com/StarRocks/ranger/blob/master/plugin-starrocks/conf/ranger-starrocks-security.xml)，并复制内容。您必须修改以下两个参数并保存修改：
- ranger.plugin.starrocks.service.name：更改为您在步骤4中创建的StarRocks服务名称。

   - - ranger.plugin.starrocks.policy.rest.url：更改为 Ranger Admin 的地址。
   - 如果需要修改其他配置，请参考 Apache Ranger 官方文档。例如，您可以修改 ranger.plugin.starrocks.policy.pollIntervalMs 来改变拉取策略变更的间隔时间。
   将配置项 access_control = ranger 添加到所有 FE 的配置文件中。

   ```SQL
   vim ranger-starrocks-security.xml
   
   ...
       <property>
           <name>ranger.plugin.starrocks.service.name</name>
           <value>starrocks</value> -- Change it to the StarRocks Service name.
           <description>
               Name of the Ranger service containing policies for this StarRocks instance
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

7. 重启所有 FE 机器。

   ```SQL
   vim fe.conf
   access_control=ranger 
   ```

8. Restart all FE machines.

   ```SQL
   -- Switch to the FE folder. 
   cd..
   
   bin/stop_fe.sh
   bin/start_fe.sh
   ```

## 重用其他服务以控制对外部表的访问

- 对于 External Catalog，您可以复用外部服务（例如 Hive 服务）进行访问控制。StarRocks 支持为不同的 Catalog 匹配不同的 Ranger 外部服务。当用户访问外部表时，系统将根据外部表对应的 Ranger 服务的访问策略执行访问控制。

1. - 将 Hive 的 Ranger 配置文件 ranger-hive-security.xml 复制到所有 FE 机器的 fe/conf 目录。
2. - 重启所有 FE 机器。
3. - 配置 External Catalog 时，添加属性 "ranger.plugin.hive.service.name"。

-    - 您也可以将此属性添加到现有的 External Catalog 中。

     ```SQL
       CREATE EXTERNAL CATALOG hive_catalog_1
       PROPERTIES (
           "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
           "ranger.plugin.hive.service.name" = "hive_catalog_1"
       )
     ```

-      这将改变现有 Catalog 的认证方法为基于 Ranger 的认证。

     ```SQL
     ALTER CATALOG hive_catalog_1
     SET ("ranger.plugin.hive.service.name" = "hive_catalog_1");
     ```

​    This operation changes the authentication method of an existing Catalog to Ranger-based authentication.

## 下一步

- 添加 StarRocks 服务后，您可以点击该服务创建访问控制策略，并为不同的用户或用户组分配不同权限。当用户访问 StarRocks 数据时，将根据这些策略执行访问控制。

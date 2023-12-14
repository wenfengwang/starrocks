---
displayed_sidebar: "Chinese"
---

# 降级 StarRocks

This article describes how to downgrade your StarRocks cluster.

If there are exceptions after upgrading the StarRocks cluster, you can downgrade it to the previous version to quickly restore the cluster.

## Overview

Please review the information in this section before downgrading. It is recommended that you downgrade the cluster according to the recommended operations in this document.

### Downgrade Paths

- **Minor Version Downgrade**

  You can downgrade your StarRocks cluster across minor versions, for example, from v2.2.11 directly to v2.2.6.

- **Major Version Downgrade**

  For compatibility and security reasons, we strongly recommend that you downgrade the StarRocks cluster **gradually across major versions**. For example, to downgrade a StarRocks v2.5 cluster to v2.2, you need to downgrade in the following order: v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x.

- **Significant Version Downgrade**

  - You cannot downgrade directly to v1.19, you must first downgrade to v2.0.
  - You can only downgrade the cluster from v3.0 to version v2.5.3 or higher.
    - StarRocks upgraded the BDB library in version v3.0. Since BDB JE cannot be rolled back, you must continue to use the v3.0 BDB library after downgrading.
    - After upgrading to v3.0, the cluster defaults to using the new RBAC permission system. After downgrading, you can only use the RBAC permission system.

### Downgrade Process

The downgrade process of StarRocks is the opposite of the [upgrade process](../deployment/upgrade.md#upgrade-process). Therefore, **you need to downgrade FE first, and then BE and CN**. Incorrect downgrade order may result in incompatibility between FE and BE/CN, leading to service crashes. For FE nodes, you must first downgrade all Follower FE nodes, and finally downgrade the Leader FE nodes.

## Preparations

During the preparation process, if you need to perform a major or significant version downgrade, you must perform compatibility configuration. Before downgrading all nodes in the cluster, you also need to perform the downgrade correctness test on one of the FE and BE nodes.

### Compatibility Configuration

If you need to perform a major or significant version downgrade, you must perform compatibility configuration. In addition to the general compatibility configuration, specific configurations based on the version before the downgrade are also required.

- **General Compatibility Configuration**

Before downgrading, please disable Tablet Clone.

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

After the downgrade is complete and all BE nodes' status becomes `Alive`, you can re-enable Tablet Clone.

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **If you are downgrading from v2.2 or later versions**

Set the FE configuration item `ignore_unknown_log_id` to `true`. Since this configuration item is a static parameter, you must modify it in the FE configuration file **fe.conf** and restart the node after the modification to take effect. After the downgrade and the first Checkpoint are completed, you can reset it to `false` and restart the node.

- **If you have enabled FQDN access**

If you have enabled FQDN access (supported since v2.4), and need to downgrade to a version before v2.4, you must switch to IP address access before the downgrade. For detailed instructions, please refer to [Roll Back FQDN](../administration/enable_fqdn.md#roll-back).

## Downgrade FE

After passing the downgrade correctness test, you can start by downgrading the FE nodes. You must first downgrade the Follower FE nodes, and then downgrade the Leader FE nodes.

1. Go to the working directory of the FE node and stop the node.

   ```Bash
   # Replace <fe_dir> with the deployment directory of the FE node.
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. Replace the original deployment file paths **bin**, **lib**, and **spark-dpp** with the deployment files of the old version.

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

   > **Note**
   >
   > If you need to downgrade from StarRocks v3.0 to v2.5, you must perform the following steps after replacing the deployment files:
   >
   > 1. Copy the **fe/lib/starrocks-bdb-je-18.3.13.jar** file in the v3.0 deployment to the **fe/lib** path of the v2.5 deployment.
   > 2. Delete the file **fe/lib/je-7.\*.jar**.

3. Start the FE node.

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. Check if the node has started successfully.

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. Repeat the above steps to downgrade the other Follower FE nodes, and finally downgrade the Leader FE node.

   > **Note**
   >
   > If you need to downgrade from StarRocks v3.0 to v2.5, you must perform the following steps after the downgrade is complete:
   >
   > 1. Execute [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) to create a new metadata snapshot file.
   > 2. Wait for the metadata snapshot file to be synchronized to other FE nodes.
   >
   > If you do not run this command, some downgrade operations may fail. The ALTER SYSTEM CREATE IMAGE command is only supported in v2.5.3 and higher versions.

## Downgrade BE

After downgrading all FE nodes, you can proceed to downgrade the BE nodes.

1. Go to the working directory of the BE node and stop the node.

   ```Bash
   # Replace <be_dir> with the deployment directory of the BE node.
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. Replace the original deployment file paths **bin** and **lib** with the deployment files of the old version.

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. Start the BE node.

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. Check if the node has started successfully.

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 降级其他 BE 节点的步骤如上所述。

## 降级 CN

1. 进入 CN 节点的工作目录，并以优雅的方式停止该节点。

   ```Bash
   # 用 CN 的部署目录替换 <cn_dir>。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 将部署文件的原路径 **bin** 和 **lib** 替换为新版本的部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. 启动该 CN 节点。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. 确认节点是否成功启动。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 重复以上步骤降级其他 CN 节点。
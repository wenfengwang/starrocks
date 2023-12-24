---
displayed_sidebar: English
---

# 安装插件

## 描述

此语句用于安装插件。

:::提示

此操作需要 SYSTEM 级 PLUGIN 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```sql
INSTALL PLUGIN FROM [source] [PROPERTIES ("key"="value", ...)]
```

支持 3 种类型的源：

```plain text
1. 直接指向 zip 文件的绝对路径
2. 直接指向插件目录的绝对路径
3. 直接指向 zip 文件的 http 或 https 下载链接
```

PROPERTIES 支持设置插件的一些配置，比如设置 zip 文件的 md5sum 值等。

## 例子

1. 从本地 zip 文件安装插件：

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo.zip";
    ```

2. 从本地目录安装插件：

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo/";
    ```

3. 下载并安装插件：

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
    ```

4. 下载并安装插件。同时，设置 zip 文件的 md5sum 值：

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
    ```

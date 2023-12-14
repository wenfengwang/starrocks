---
displayed_sidebar: "Chinese"
---

# 安装插件

## 描述

此语句用于安装插件，

语法：

```sql
INSTALL PLUGIN FROM [source] [PROPERTIES ("key"="value", ...)]
```

支持3种类型的来源：

```plain text
1. 指向zip文件的绝对路径
2. 指向插件目录的绝对路径
3. 指向zip文件的http或https下载链接
```

PROPERTIES支持设置插件的一些配置，如设置zip文件的md5sum值等。

## 示例

1. 从本地zip文件安装插件：

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo.zip";
    ```

2. 从本地inpath安装插件：

    ```sql
    INSTALL PLUGIN FROM "/home/users/starrocks/auditdemo/";
    ```

3. 下载并安装插件：

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
    ```

4. 下载并安装插件。同时，设置zip文件的md5sum值：

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
    ```
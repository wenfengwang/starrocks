---
displayed_sidebar: English
---

# 使用 debuginfo 文件进行调试

## 更改说明

从 v2.5 版本开始，BE 的 debuginfo 文件已从 StarRocks 安装包中移除，目的是为了减小安装包的体积和减少空间占用。您可以在 [StarRocks 官网](https://www.starrocks.io/download/community) 上看到两种不同的安装包。

![debuginfo](../assets/debug_info.png)

在该图示中，您可以点击 `获取调试符号文件` 来下载 debuginfo 包。`StarRocks-2.5.10.tar.gz` 是安装包，您可以点击 **下载** 来获取这个安装包。

此变更不会影响您下载或使用 StarRocks 的方式。您可以仅下载安装包进行集群部署和升级。debuginfo 包只适用于开发者使用 GDB 进行程序调试。

## 注意事项

建议使用 GDB 12.1 版本或更新的版本来进行调试。

## 如何使用 debuginfo 文件

1. 下载并解压 debuginfo 包。

   ```SQL
   wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz
   
   tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
   ```

      > **注意**
      > 请将 `\u003csr_ver\u003e` 替换为您想要下载的 StarRocks 安装包的版本号。

2. 在进行 GDB 调试时，加载 debuginfo 文件。

   -  **方法一**

   ```Shell
   objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
   ```

   此操作会将调试信息文件与您的可执行文件关联起来。

   -  **方法二**

   ```Shell
   gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
   ```

debuginfo 文件可与 perf 和 pstack 良好配合。您可以直接使用 perf 和 pstack，无需进行额外的操作。

---
displayed_sidebar: English
---

# 使用 debuginfo 文件进行调试

## 更改说明

从 v2.5 版本开始，StarRocks 安装包中移除了 BE 的 debuginfo 文件，以减少安装包的大小和空间占用。您可以在 [StarRocks 官网](https://www.starrocks.io/download/community) 上看到两个包。

![debuginfo](../assets/debug_info.png)

在此图中，您可以点击 `获取调试符号文件` 来下载 debuginfo 包。`StarRocks-2.5.10.tar.gz` 是安装包，您可以点击 **下载** 来下载该安装包。

此更改不会影响您下载或使用 StarRocks 的行为。您可以仅下载安装包进行集群部署和升级。debuginfo 包仅供开发者使用 GDB 进行程序调试。

## 注意事项

推荐使用 GDB 12.1 或更高版本进行调试。

## 如何使用 debuginfo 文件

1. 下载并解压 debuginfo 包。

   ```SQL
   wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz
   
   tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
   ```

      > **注意**
      > 将 `<sr_ver>` 替换为您想要下载的 StarRocks 安装包的版本号。

2. 在进行 GDB 调试时加载 debuginfo 文件。

   -  **方法一**

   ```Shell
   objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
   ```

   此操作会将 debuginfo 文件与您的可执行文件关联起来。

   -  **方法二**

   ```Shell
   gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
   ```

debuginfo 文件与 perf 和 pstack 兼容良好。您可以直接使用 perf 和 pstack，无需进行额外操作。
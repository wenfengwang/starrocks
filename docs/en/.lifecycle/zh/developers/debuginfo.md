---
displayed_sidebar: English
---

# 使用 debuginfo 文件进行调试

## 更改说明

从 v2.5 版本开始，BE 的 debuginfo 文件已经从 StarRocks 安装包中剥离，以减少安装包的大小和空间占用。您可以在 [StarRocks 网站](https://www.starrocks.io/download/community) 上找到两个软件包。

![调试信息](../assets/debug_info.png)

在这个图中，您可以点击 `获取调试符号文件` 来下载 debuginfo 包。`StarRocks-2.5.10.tar.gz` 是安装包，您可以点击 **下载** 来下载这个包。

这个变化不会影响您下载或使用 StarRocks。您可以只下载安装包来部署和升级集群。debuginfo 包仅供开发人员使用 GDB 来调试程序。

## 注意事项

建议使用 GDB 12.1 或更高版本进行调试。

## 如何使用 debuginfo 文件

1. 下载并解压 debuginfo 包。

    ```SQL
    wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz

    tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
    ```

    > **注意**
    >
    > 将 `<sr_ver>` 替换为您想要下载的 StarRocks 安装包的版本号。

2. 在进行 GDB 调试时加载 debuginfo 文件。

    - **方法 1**

    ```Shell
    objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
    ```

    这个操作会将调试信息文件与您的可执行文件相关联。

    - **方法 2**

    ```Shell
    gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
    ```

debuginfo 文件可以很好地配合 perf 和 pstack 使用。您可以直接使用 perf 和 pstack 而无需额外操作。

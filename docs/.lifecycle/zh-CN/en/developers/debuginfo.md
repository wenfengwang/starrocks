---
displayed_sidebar: "Chinese"
---

# 使用debuginfo文件进行调试

## 修改说明

从v2.5开始，BE的debuginfo文件已经从StarRocks安装包中剥离，以减小安装包的大小和空间占用。您可以在[StarRocks网站](https://www.starrocks.io/download/community)上看到两个包。

![debuginfo](../assets/debug_info.png)

在此图中，您可以单击“获取Debug符号文件”以下载debuginfo包。“StarRocks-2.5.10.tar.gz”是安装包，您可以单击 **Download** 以下载此包。

这一变更不会影响您下载或使用StarRocks的行为。您可以仅下载安装包进行集群部署和升级。debuginfo包仅供开发人员使用GDB调试程序。

## 注意事项

建议使用GDB 12.1或更高版本进行调试。

## 如何使用debuginfo文件

1. 下载并解压debuginfo包。

    ```SQL
    wget https://releases.starrocks.io/starrocks/StarRocks-<sr_ver>.debuginfo.tar.gz

    tar -xzvf StarRocks-<sr_ver>.debuginfo.tar.gz
    ```

    > **注意**
    >
    > 使用要下载的StarRocks安装包的版本号替换 `<sr_ver>`。

2. 在执行GDB调试时加载debuginfo文件。

    - **方法1**

    ```Shell
    objcopy --add-gnu-debuglink=starrocks_be.debug starrocks_be
    ```

    此操作将调试信息文件与您的可执行文件关联起来。

    - **方法2**

    ```Shell
    gdb -s starrocks_be.debug -e starrocks_be -c `core_file`
    ```

debuginfo文件可以与perf和pstack很好地配合使用。您可以直接使用perf和pstack而无需额外操作。
---
displayed_sidebar: English
---

# 在 IDEA 上设置 StarRocks FE 开发环境

本教程基于 macOS，并已在 Apple Chip（M1、M2）上进行了测试。
即使您没有使用 macOS，您也可以参考本教程。

## 要求

### Thrift 0.13

官方 brew 存储库中没有 0.13 版本的 Thrift；我们的一位贡献者在其存储库中创建了一个版本进行安装。

```bash
brew install alberttwong/thrift/thrift@0.13
```

成功安装 Thrift 后，您可以通过执行以下命令进行检查：

```bash
$ thrift -version
Thrift version 0.13.0
```

### Protobuf

只需使用最新的 v3 版本，因为最新版本的 Protobuf 兼容 StarRocks 中使用的 v2 版本的 Protobuf。

```bash
brew install protobuf
```

### Maven

```
brew install maven
```

### Openjdk 1.8 或 11

```bash
brew install openjdk@11
```

### Python3

默认情况下，MacOS 已经安装了 Python3。


每个人的 Thrift 和 Protobuf 安装目录可能不同，您可以使用 brew list 命令进行检查：

```bash
brew list thrift@0.13.0
brew list protobuf
```

## 配置 StarRocks

### 下载 StarRocks

```
git clone https://github.com/StarRocks/starrocks.git
```

### 设置 thirdparty 目录

在 `thirdparty` 目录中创建 `installed/bin` 目录。

```bash
cd starrocks && mkdir -p thirdparty/installed/bin
```

然后分别为 Thrift 和 Protobuf 创建软链接。

```bash
ln -s /opt/homebrew/bin/thrift thirdparty/installed/bin/thrift
ln -s /opt/homebrew/bin/protoc thirdparty/installed/bin/protoc
```

### 设置环境变量

```bash
export JAVA_HOME="/opt/homebrew/Cellar/openjdk@11/11.0.15" # 注意：您的 JDK 版本可能与桌面上的不同
export PYTHON=/usr/bin/python3
export STARROCKS_THIRDPARTY=$(pwd)/thirdparty # 注意：确保您在 starrocks 目录中
```

## 生成源代码

FE 中的许多源文件需要手动生成，否则 IDEA 会因缺少文件而报错。
执行以下命令自动生成：

```bash
cd gensrc
make clean
make
```

## 编译 FE

进入 `fe` 目录，使用 Maven 进行编译：

```bash
cd fe
mvn install -DskipTests
```

## 在 IDEA 中打开 StarRocks

1. 在 IDEA 中打开 `StarRocks` 目录。

2. 添加编码样式设置
    为了规范编码风格，您应该在 IDEA 中导入 `fe/starrocks_intellij_style.xml` 代码样式文件。
![image-20220701193938856](../../assets/IDEA-2.png)

## 在 MacOS 中运行 StarRocks FE

使用 IDEA 打开 `fe` 目录。

如果直接在 `StarRocksFE.java` 中执行 Main 函数，将会报告一些错误。您只需进行一些简单的设置即可顺利运行。

**注意：** `StarRocksFE.java` 位于 `fe/fe-core/src/main/java/com/starrocks` 目录中。

1. 将 StarRocks 目录下的 conf、bin 和 webroot 目录复制到 `fe` 目录：

```bash
cp -r conf fe/conf
cp -r bin fe/bin
cp -r webroot fe/webroot
```

2. 进入 `fe` 目录，在其中创建 log 和 meta 文件夹：

```bash
cd fe
mkdir log
mkdir meta
```

3. 设置环境变量，如下图所示：

![image-20220701193938856](../../assets/IDEA-1.png)

```bash
export PID_DIR=/Users/smith/Code/starrocks/fe/bin
export STARROCKS_HOME=/Users/smith/Code/starrocks/fe
export LOG_DIR=/Users/smith/Code/starrocks/fe/log
```

4. 修改 `fe/conf/fe.conf` 中的 `priority_networks` 为 `127.0.0.1/24`，以防止 FE 使用当前计算机的局域网 IP 导致端口绑定失败。

5. 然后您就成功运行了 StarRocks FE。

## 在 MacOS 中调试 StarRocks FE

如果您使用 debug 选项启动了 FE，则可以将 IDEA 调试器附加到 FE 进程。


```
./start_fe.sh --debug
```

请参阅 https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local。

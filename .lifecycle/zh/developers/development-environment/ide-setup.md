---
displayed_sidebar: English
---

# 设置用于开发 StarRocks 的 IDE

很多人想成为 StarRocks 的贡献者，但开发环境的搭建让他们感到困扰，因此我在这里撰写了一篇教程。

什么是理想的开发工具链？

* 支持一键编译 FE（前端）和 BE（后端）。
* 支持在 Clion 和 IDEA 中进行代码跳转。
* IDE 中的所有变量都能正常解析，不会出现红色波浪线。
* Clion 能够正常启用其分析功能。
* 支持 FE 和 BE 的调试。

## 准备工作

我使用 MacBook（M1）进行本地编码，并使用远程服务器编译及测试 StarRocks。（远程服务器采用 Ubuntu 22，**至少需要 16GB RAM**）。

总体思路是在 MacBook 上编写代码，然后通过 IDE 自动将代码同步到服务器，使用服务器来编译和开发 StarRocks。

### MacBook 配置

#### Thrift 0.13

官方 brew 仓库中没有 0.13 版本的 Thrift；我们的一位贡献者在他们的仓库中创建了一个版本供安装。

```bash
brew install alberttwong/thrift/thrift@0.13
```

你可以通过以下命令检查 Thrift 是否安装成功：

```bash
$ thrift -version
Thrift version 0.13.0
```

#### Protobuf

直接使用最新的 v3 版本，因为 StarRocks 中的 v2 版本 Protobuf 协议与最新版的 Protobuf 兼容。

```bash
brew install protobuf
```

#### Maven

```bash
brew install maven
```

#### OpenJDK 1.8 或 11

```bash
brew install openjdk@11
```

#### Python3

MacOS 自带，无需安装。

#### 设置系统环境变量

```bash
export JAVA_HOME=xxxxx
export PYTHON=/usr/bin/python3
```

### Ubuntu 22 服务器配置

#### 克隆 StarRocks 代码

git clone https://github.com/StarRocks/starrocks.git

#### 安装编译所需工具

```bash
sudo apt update
```

```bash
sudo apt install gcc g++ maven openjdk-11-jdk python3 python-is-python3 unzip cmake bzip2 ccache byacc ccache flex automake libtool bison binutils-dev libiberty-dev build-essential ninja-build
```

设置 JAVA_HOME 环境变量

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

#### 编译 StarRocks

```bash
cd starrocks/
./build.sh
```

首次编译需要编译第三方依赖库，这将花费一些时间。

**首次编译必须使用 gcc**，因为目前第三方依赖库在 clang 中无法成功编译。

## IDE 配置

### FE

FE 开发相对简单，因为你可以直接在 MacOS 上编译。只需进入 fe 目录并运行命令 mvn install -DskipTests。

然后可以直接使用 IDEA 打开 fe 目录，一切就绪。

#### 本地调试

与其他 Java 应用程序相同。

#### 远程调试

在 Ubuntu 服务器上，运行 ./start_fe.sh --debug，然后使用 IDEA 的远程调试功能连接。默认端口是 5005，你可以在 start_fe.sh 脚本中修改它。

调试 Java 参数：-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005，这个参数直接从 IDEA 中复制即可。

![IDE](../../assets/ide-1.png)

### BE

建议首先在 fe 目录下运行 mvn install -DskipTests，确保 gensrc 目录下的 thrift 和 protobuf 能够正确编译。

然后需要进入 gensrc 目录，依次运行 make clean 和 make 命令，否则 Clion 无法检测到 thrift 生成的文件。

使用 Clion 打开 be 目录。

进入设置，添加工具链。首先添加远程服务器，然后分别设置构建工具、C 和 C++ 编译器。

![IDE](../../assets/ide-2.png)

在设置/部署中，更改文件夹映射。

![IDE](../../assets/ide-3.png)

在设置/CMake 中，将工具链更改为刚刚添加的远程工具链。添加以下环境变量：

```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
STARROCKS_GCC_HOME=/usr/
STARROCKS_THIRDPARTY=/root/starrocks/thirdparty
```

注意：确保不要勾选“包含系统环境变量”。

![IDE](../../assets/ide-4.png)

![IDE](../../assets/ide-5.png)

至此，所有设置均已完成。Clion 和远程服务器同步一段时间后，代码跳转功能将正常工作。

#### 调试

BE 调试有些复杂，你必须在远程服务器上使用 gdb。当然，你也可以使用 gdb 服务器 + Clion 的远程 gdb 功能，但我不推荐这么做，因为体验不佳。

我们需要将 start_backend.sh 脚本从：

```bash
if [ ${RUN_BE} -eq 1 ]; then
    echo "start time: "$(date) >> $LOG_DIR/be.out
    if [ ${RUN_DAEMON} -eq 1 ]; then
        nohup ${STARROCKS_HOME}/lib/starrocks_be "$@" >> $LOG_DIR/be.out 2>&1 </dev/null &
    else
        ${STARROCKS_HOME}/lib/starrocks_be "$@" >> $LOG_DIR/be.out 2>&1 </dev/null
    fi
fi
```

更改为：

```bash
if [ ${RUN_BE} -eq 1 ]; then
    echo "start time: "$(date) >> $LOG_DIR/be.out
    if [ ${RUN_DAEMON} -eq 1 ]; then
        nohup ${STARROCKS_HOME}/lib/starrocks_be "$@" >> $LOG_DIR/be.out 2>&1 </dev/null &
    else
        gdb -tui ${STARROCKS_HOME}/lib/starrocks_be
    fi
fi
```

然后直接运行 ./bin/start_be.sh，不需要任何参数。

> 如果在调试 lakehouse 时遇到错误报告，只需在 ~/.gdbinit 中添加 handle SIGSEGV nostop noprint pass 即可。

#### LLVM

当然，你也可以使用 LLVM 工具来开发 BE。

Ubuntu LLVM 安装请参考：https://apt.llvm.org/

然后使用命令：CC=clang-15 CXX=clang++-15 ./build.sh 来编译 BE。但前提是你的第三方依赖库已经用 gcc 编译过。

## 最后

随时欢迎向 StarRocks 贡献代码。🫵

## 参考资料

* [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en)
* 中文版：[https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env)

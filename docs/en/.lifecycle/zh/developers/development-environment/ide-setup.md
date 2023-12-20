---
displayed_sidebar: English
---

# 设置用于开发 StarRocks 的 IDE

有些人想成为 StarRocks 贡献者，但受到开发环境的困扰，所以在这里我写了一篇教程。

什么是完美的开发工具链？

* 支持一键编译 FE 和 BE。
* 支持 Clion 和 IDEA 中的代码跳转。
* IDE 中所有变量都可以正常分析，没有红线。
* Clion 可以正常启用其分析功能。
* 支持 FE 和 BE 调试。

## 准备

我使用 MacBook(M1) 进行本地编码，并使用远程服务器进行 StarRocks 的编译和测试。（远程服务器使用 Ubuntu 22，**至少需要 16GB RAM**）。

总体思路是在 MacBook 上编写代码，然后通过 IDE 自动将代码同步到服务器，并使用服务器来编译和开发 StarRocks。

### MacBook 设置

#### Thrift 0.13

官方 brew 仓库中没有 0.13 版本的 Thrift；我们的一位提交者在他们的仓库中创建了一个版本来安装。

```bash
brew install alberttwong/thrift/thrift@0.13
```

可以通过以下命令检查 Thrift 是否安装成功：

```bash
$ thrift -version
Thrift version 0.13.0
```

#### Protobuf

直接使用最新版本 v3，因为最新版本的 Protobuf 与 StarRocks 中 v2 版本的 Protobuf 协议兼容。

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

### Ubuntu 22 服务器设置

#### 克隆 StarRocks 代码

`git clone https://github.com/StarRocks/starrocks.git`

#### 安装编译所需工具

```bash
sudo apt update
```

```bash
sudo apt install gcc g++ maven openjdk-11-jdk python3 python-is-python3 unzip cmake bzip2 ccache byacc ccache flex automake libtool bison binutils-dev libiberty-dev build-essential ninja-build
```

设置 `JAVA_HOME` 环境变量

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

#### 编译 StarRocks

```bash
cd starrocks/
./build.sh
```

首次编译需要编译 thirdparty，这将需要一些时间。

**首次编译必须使用 gcc，因为目前 thirdparty 无法在 clang 中成功编译。**

## IDE 设置

### FE

FE 开发很简单，因为你可以直接在 MacOS 中编译它。只需进入 `fe` 文件夹并运行命令 `mvn install -DskipTests`。

然后你可以直接使用 IDEA 打开 `fe` 文件夹，一切就绪。

#### 本地调试

与其他 Java 应用程序相同。

#### 远程调试

在 Ubuntu 服务器上，运行 `./start_fe.sh --debug`，然后使用 IDEA 远程调试连接。默认端口是 5005，你可以在 `start_fe.sh` 脚本中更改它。

调试 Java 参数：`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005` 是直接从 IDEA 复制的。

![IDE](../../assets/ide-1.png)

### BE

建议先在 `fe` 文件夹下运行 `mvn install -DskipTests`，以确保 `gensrc` 目录下的 thrift 和 protobuf 正确编译。

然后需要进入 `gensrc` 文件夹，分别运行 `make clean` 和 `make` 命令，否则 Clion 无法检测 thrift 的输出文件。

使用 Clion 打开 `be` 文件夹。

进入 `Settings`，添加 `Toolchains`。首先添加远程服务器，然后分别设置 Build Tool、C 和 C++ 编译器。

![IDE](../../assets/ide-2.png)

在 `Settings` / `Deployment` 中。更改文件夹 `mappings`。

![IDE](../../assets/ide-3.png)

在 `Settings` / `CMake` 中。将 Toolchain 更改为刚刚添加的远程工具链。添加以下环境变量：

```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
STARROCKS_GCC_HOME=/usr/
STARROCKS_THIRDPARTY=/root/starrocks/thirdparty
```

注意：确保不要勾选 `Include system environment variables`。

![IDE](../../assets/ide-4.png)

![IDE](../../assets/ide-5.png)

从这里开始，所有设置就完成了。Clion 和远程服务器同步一段时间后，代码跳转就可以正常工作了。

#### 调试

BE 调试有点困难，你必须在远程服务器上使用 gdb。当然，你可以使用 gdb server + Clion 远程 gdb，但我不推荐，体验不佳。

我们需要将 `start_backend.sh` 脚本从：

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

改为：

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

然后只需运行 `./bin/start_be.sh` 而不带任何参数。

> 如果在调试 lakehouse 时遇到错误报告，只需在 `~/.gdbinit` 中添加 `handle SIGSEGV nostop noprint pass`。

#### LLVM

当然，你也可以使用 LLVM 工具进行 BE 开发。

Ubuntu LLVM 安装参考：https://apt.llvm.org/

然后使用命令：`CC=clang-15 CXX=clang++-15 ./build.sh` 来编译 BE。但前提是你的 thirdparty 已经用 gcc 编译过。

## 最后

欢迎向 StarRocks 贡献代码。🫵

## 参考

* [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en)
* 中文版：[https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env)
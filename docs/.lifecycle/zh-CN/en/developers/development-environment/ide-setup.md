---
displayed_sidebar: "Chinese"
---

# 为StarRocks开发设置IDE

一些人希望成为StarRocks的贡献者，但受到开发环境的困扰，所以我在这里撰写了一篇教程。

什么是完善的开发工具链？

* 支持一键编译FE和BE。
* 支持在Clion和IDEA中跳转到代码。
* IDE中的所有变量都能正常分析，没有红色波浪线。
* Clion可以正常启用其分析功能。
* 支持FE和BE调试。

## 准备

我在本地使用MacBook(M1)进行编码，远程服务器用于编译和测试StarRocks（远程服务器使用Ubuntu 22，**至少需要16GB RAM**）。

整体想法是在MacBook上编写代码，然后通过IDE自动将代码同步到服务器，并使用服务器编译和开发StarRocks。

### MacBook设置

#### Thrift 0.13

官方brew库中没有0.13版本的Thrift；我们的一位贡献者在其repo中创建了一个版本以进行安装。

```bash
brew install alberttwong/thrift/thrift@0.13
```

您可以使用以下命令检查Thrift是否安装成功：

```bash
$ thrift -version
Thrift version 0.13.0
```

#### Protobuf

直接使用最新版本v3，因为最新版本的Protobuf兼容StarRocks中的v2版本的Protobuf协议。

```bash
brew install protobuf
```

#### Maven

```bash
brew install maven
```

#### OpenJDK 1.8或11

```bash
brew install openjdk@11
```

#### Python3

MacOS自带，无需安装。

#### 设置系统环境变量

```bash
export JAVA_HOME=xxxxx
export PYTHON=/usr/bin/python3
```

### Ubuntu22服务器设置

#### 克隆StarRocks代码

`git clone https://github.com/StarRocks/starrocks.git`

#### 安装编译所需工具

```bash
sudo apt update
```

```bash
sudo apt install gcc g++ maven openjdk-11-jdk python3 python-is-python3 unzip cmake bzip2 ccache byacc ccache flex automake libtool bison binutils-dev libiberty-dev build-essential ninja-build
```

设置`JAVA_HOME`环境变量

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

#### 对StarRocks进行编译

```bash
cd starrocks/
./build.sh
```

第一次编译需要编译thirdparty，这将需要一些时间。

**您必须在第一次编译时使用gcc，目前thirdparty无法在clang中成功编译。**

## IDE设置

### FE

FE的开发很简单，因为可以直接在MacOS中编译它。只需进入`fe`文件夹，运行命令`mvn install -DskipTests`。

然后您可以使用IDEA直接打开`fe`文件夹，一切正常。

#### 本地调试

与其他Java应用程序一样。

#### 远程调试

在Ubuntu服务器上，使用`./start_fe.sh --debug`运行，然后使用IDEA远程调试连接它。默认端口是5005，您可以在`start_fe.sh`脚本中更改它。

调试java参数：`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005`只是从IDEA中复制过来的。

![IDE](../../assets/ide-1.png)

### BE

建议先在`fe`文件夹中运行`mvn install -DskipTests`，以确保gensrc目录中的thrift和protobuf编译正确。

然后您需要进入`gensrc`文件夹，分别运行`make clean`和`make`命令，否则Clion无法检测到thrift的输出文件。

使用Clion打开`be`文件夹。

进入`设置`，添加`工具链`。首先添加一个远程服务器，然后分别设置构建工具、C和C++编译器。

![IDE](../../assets/ide-2.png)

在`设置` / `部署`中。更改文件夹`映射`。

![IDE](../../assets/ide-3.png)

在`设置` / `Cmake`中。将工具链更改为刚刚添加的远程工具链。添加以下环境变量：

```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
STARROCKS_GCC_HOME=/usr/
STARROCKS_THIRDPARTY=/root/starrocks/thirdparty
```

注意：小心不要勾选`包括系统环境变量`。

![IDE](../../assets/ide-4.png)

![IDE](../../assets/ide-5.png)

从这里开始，所有设置就完成了。在Clion和远程服务器同步一段时间后，代码跳转将正常工作。

#### 调试

BE的调试有些困难，您必须在远程服务器上使用gdb。当然，您可以使用gdb服务器+Clion远程gdb，但我不建议这样做，太卡了。

我们需要将`start_backend.sh`脚本从：

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

然后只需运行`./bin/start_be.sh`，不带任何标志。

> 如果在对lakehouse进行调试时遇到错误报告，只需在`~/.gdbinit`中添加 `handle SIGSEGV nostop noprint pass`。

#### LLVM

当然，您可以使用LLVM工具进行BE开发。

Ubuntu中LLVM的安装参考：https://apt.llvm.org/

然后使用命令：`CC=clang-15 CXX=clang++-15 ./build.sh` 编译be。但前提是您的thirdparty已经用gcc编译过。

## 最后

欢迎为StarRocks贡献代码。🫵

## 参考

* [https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env-en)
* 中文版：[https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env](https://www.inlighting.org/archives/setup-perfect-starrocks-dev-env)
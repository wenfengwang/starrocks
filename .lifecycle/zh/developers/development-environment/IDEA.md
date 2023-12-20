---
displayed_sidebar: English
---

# 在IDEA上配置StarRocks FE开发环境

本教程基于macOS系统，并已在Apple芯片（M1、M2）上完成测试。即便您不使用macOS系统，本教程也能提供参考。

## 环境要求

### Thrift 0.13

官方brew仓库中不存在0.13版本的Thrift；我们的一位贡献者在他们的仓库中创建了一个版本供安装。

```bash
brew install alberttwong/thrift/thrift@0.13
```

成功安装Thrift后，可以执行以下命令来检查：

```bash
$ thrift -version
Thrift version 0.13.0
```

### Protocol Buffers

直接使用最新的v3版本，因为最新的Protobuf版本与StarRocks中使用的v2版本的Protobuf兼容。

```bash
brew install protobuf
```

### Maven

```
brew install maven
```

### OpenJDK 1.8 或 11

```bash
brew install openjdk@11
```

### Python3

MacOS系统默认已安装。

每个人安装Thrift和Protobuf的目录可能不同，您可以使用brew list命令来查看：

```bash
brew list thrift@0.13.0
brew list protobuf
```

## 配置StarRocks

### 下载StarRocks

```
git clone https://github.com/StarRocks/starrocks.git
```

### 设置thirdparty目录

在thirdparty目录中创建installed/bin文件夹。

```bash
cd starrocks && mkdir -p thirdparty/installed/bin
```

然后分别为Thrift和Protobuf创建软链接。

```bash
ln -s /opt/homebrew/bin/thrift thirdparty/installed/bin/thrift
ln -s /opt/homebrew/bin/protoc thirdparty/installed/bin/protoc
```

### 设置环境变量

```bash
export JAVA_HOME="/opt/homebrew/Cellar/openjdk@11/11.0.15" # Caution: The jdk version may be different in you desktop
export PYTHON=/usr/bin/python3
export STARROCKS_THIRDPARTY=$(pwd)/thirdparty # Caution: Make sure you are in the starrocks directory
```

## 生成源代码

FE中有许多源文件需要手动生成，否则IDEA会因缺少文件而报错。执行以下命令以自动生成：

```bash
cd gensrc
make clean
make
```

## 编译FE

进入fe目录，使用Maven进行编译：

```bash
cd fe
mvn install -DskipTests
```

## 在IDEA中打开StarRocks

1. 在IDEA中打开StarRocks目录。

2. 添加代码风格设置
为了规范代码风格，你需要在IDEA中导入`fe/starrocks_intellij_style.xml`代码风格文件。
![image-20220701193938856](../../assets/IDEA-2.png)

## 在MacOS中运行StarRocks FE

使用IDEA打开fe目录。

如果在StarRocksFE.java中直接执行Main函数，可能会报错。您只需进行一些简单的设置即可顺利运行。

**注意：** `StarRocksFE.java` 位于 `fe/fe-core/src/main/java/com/starrocks` 目录下。

1. 从StarRocks目录中复制conf、bin和webroot目录到fe目录：

```bash
cp -r conf fe/conf
cp -r bin fe/bin
cp -r webroot fe/webroot
```

2. 进入fe目录，在fe目录下创建log和meta文件夹：

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

4. 修改fe/conf/fe.conf中的priority_networks为127.0.0.1/24，以防止FE使用当前计算机的局域网IP，导致端口绑定失败。

5. 至此，您已成功运行StarRocks FE。

## 在MacOS中调试StarRocks FE

如果您以调试选项启动了FE，接下来可以将IDEA的调试器连接到FE进程。

```
./start_fe.sh --debug
```

查看[https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local](https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local)。

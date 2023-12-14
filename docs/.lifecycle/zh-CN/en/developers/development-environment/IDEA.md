---
displayed_sidebar: "Chinese"
---

# 在IDEA上建立StarRocks FE开发环境

本教程是基于macOS平台的，且已经在苹果芯片(M1, M2)上进行了测试。
即使你不是用macOS，也可以参考本教程。

## 环境要求

### Thrift 0.13

官方brew仓库中没有0.13版本的Thrift；我们的一位贡献者在他们的仓库中创建了一个安装版本。

```bash
brew install alberttwong/thrift/thrift@0.13
```

成功安装Thrift后，可以通过执行以下命令来检查：

```bash
$ thrift -version
Thrift version 0.13.0
```

### Protobuf

直接使用最新版本v3，因为最新版本的Protobuf兼容于StarRocks所使用的v2版本的Protobuf。

```bash
brew install protobuf
```

### Maven

```bash
brew install maven
```

### Openjdk 1.8 or 11

```bash
brew install openjdk@11
```

### Python3

MacOS默认已安装。


每个人的Thrift和Protobuf安装目录可能不同，可以使用brew list命令来检查：

```bash
brew list thrift@0.13.0
brew list protobuf
```

## 配置StarRocks

### 下载StarRocks

```
git clone https://github.com/StarRocks/starrocks.git
```

### 配置thirdparty目录

在`thirdparty`目录下创建`installed/bin`目录。

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
export JAVA_HOME="/opt/homebrew/Cellar/openjdk@11/11.0.15" # 注意：你的jdk版本可能在你的桌面上有所不同
export PYTHON=/usr/bin/python3
export STARROCKS_THIRDPARTY=$(pwd)/thirdparty # 注意：确保你在starrocks目录下
```

## 生成源代码

FE中许多源文件需要手动生成，否则IDEA会因找不到文件而报错。
执行以下命令自动生成：

```bash
cd gensrc
make clean
make
```

## 编译FE

进入`fe`目录，使用Maven进行编译：

```bash
cd fe
mvn install -DskipTests
```

## 在IDEA中打开StarRocks

1. 在IDEA中打开`StarRocks`目录。

2. 添加编码样式设置
    为了规范化编码风格，你应该在IDEA中导入`fe/starrocks_intellij_style.xml`代码样式文件。
![image-20220701193938856](../../assets/IDEA-2.png)

## 在MacOS中运行StarRocks FE

使用IDEA打开`fe`目录。

若直接在`StarRocksFE.java`中执行Main函数，会报告一些错误。你只需做一些简单的设置即可顺利运行。

**注意：** `StarRocksFE.java`在`fe/fe-core/src/main/java/com/starrocks`目录下。

1. 从StarRocks目录中将conf、bin和webroot目录复制到`fe`目录中：

```bash
cp -r conf fe/conf
cp -r bin fe/bin
cp -r webroot fe/webroot
```

2. 进入`fe`目录，在`fe`目录下创建log和meta文件夹：

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

4. 将`fe/conf/fe.conf`中的priority_networks修改为`127.0.0.1/24`，以防止FE使用当前计算机的局域网IP导致端口无法绑定。

5. 这样你就成功运行了StarRocks FE。

## 在MacOS中调试StarRocks FE

若以调试选项启动了FE，你可以将IDEA调试器连接到FE进程。


```
./start_fe.sh --debug
```

请参阅https://www.jetbrains.com/help/idea/attaching-to-local-process.html#attach-to-local。
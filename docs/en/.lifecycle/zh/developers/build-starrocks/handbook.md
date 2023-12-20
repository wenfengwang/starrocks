---
displayed_sidebar: English
---

# 如何构建 StarRocks

一般来说，你可以通过执行以下命令来构建 StarRocks

```
./build.sh
```

该命令将首先检查所有第三方依赖项是否准备就绪。如果所有依赖项都准备就绪，它将构建 StarRocks 的 `Backend` 和 `Frontend`。

该命令成功执行后，生成的二进制文件将位于 `output` 目录中。

## 分别构建 FE/BE

你不需要每次都构建 FE 和 BE，你可以分别构建它们。例如，你可以只构建 BE
```
./build.sh --be
```

并且，只通过以下命令构建 FE
```
./build.sh --fe
```

# 如何运行单元测试

BE 和 FE 的单元测试是分开的。一般来说，你可以通过以下命令运行 BE 测试
```
./run-be-ut.sh
```

通过以下命令运行 FE 测试
```
./run-fe-ut.sh
```

## 如何在命令行中运行 BE UT

现在，BE UT 需要一些依赖才能运行，`./run-be-ut.sh` 脚本可以帮助解决这个问题。但它不够灵活。当你想在命令行中运行 UT 时，可以执行

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks Backend UT 是基于 google-test 构建的，因此你可以传递 filter 来运行部分 UT，例如，你只想测试与 MapColumn 相关的测试，可以执行

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```

# 构建选项

## 使用 clang 构建

你也可以使用 `clang` 来构建 StarRocks

```
CC=clang CXX=clang++ ./build.sh --be
```

然后你可以在构建消息中看到如下类似的信息

```
-- compiler Clang version 14.0.0
```

## 使用不同的链接器构建

默认的链接器速度较慢，开发者可以指定不同的链接器来加快链接速度。例如，你可以使用 `lld`，这是基于 LLVM 的链接器。

你需要先安装 `lld`。

```
sudo apt install lld
```

然后你可以设置环境变量 STARROCKS_LINKER 来指定你想使用的链接器。例如：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 构建不同类型

你可以通过设置不同的 BUILD_TYPE 变量来构建不同类型的 StarRocks，默认的 BUILD_TYPE 是 `RELEASE`。例如，你可以通过以下方式构建带有 `ASAN` 类型的 StarRocks：
```
BUILD_TYPE=ASAN ./build.sh --be
```
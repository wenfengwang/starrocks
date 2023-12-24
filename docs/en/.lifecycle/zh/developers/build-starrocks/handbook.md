---
displayed_sidebar: English
---

# 如何构建 StarRocks

通常情况下，您只需执行

```
./build.sh
```

此命令将首先检查所有第三方依赖项是否已准备就绪。如果所有依赖项都已准备就绪，它将构建 StarRocks 的 `Backend` 和 `Frontend`。

成功执行此命令后，生成的二进制文件将位于 `output` 目录中。

## 单独构建 FE/BE

您不需要每次都同时构建 FE 和 BE，您可以单独构建它们。
例如，您可以只构建 BE，通过以下命令
```
./build.sh --be
```

而且，只构建 FE，通过以下命令
```
./build.sh --fe
```

# 如何运行单元测试

BE 和 FE 的单元测试是分开的。通常，您可以通过以下方式运行 BE 测试
```
./run-be-ut.sh
```

运行 FE 测试 
```
./run-fe-ut.sh
```

## 如何在命令行中运行 BE UT

现在，BE UT 需要一些依赖才能运行，并且 `./run-be-ut.sh` 有所帮助。但它不够灵活。当您想在命令行中运行 UT 时，您可以执行

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks 的 Backend UT 是建立在 google-test 之上的，因此您可以通过筛选器来运行一些 UT。例如，如果您只想测试与 MapColumn 相关的测试，您可以执行

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```


# 构建选项

## 使用 Clang 构建

您也可以使用 `clang` 来构建 StarRocks

```
CC=clang CXX=clang++ ./build.sh --be
```

然后，您可以在构建消息中看到类似以下的消息

```
-- compiler Clang version 14.0.0
```

## 使用不同的链接器生成

默认的链接器速度较慢，开发者可以指定不同的链接器来加快链接速度。
例如，您可以使用 `lld`，基于 LLVM 的链接器。

您需要先安装 `lld`。

```
sudo apt install lld
```

然后，将环境变量 STARROCKS_LINKER 设置为您想要使用的链接器。
例如：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 构建不同的类型

您可以使用不同的 BUILD_TYPE 变量来构建不同类型的 StarRocks，默认的 BUILD_TYPE 是 `RELEASE`。例如，您可以通过以下方式使用 `ASAN` 类型构建 StarRocks
```
BUILD_TYPE=ASAN ./build.sh --be
```

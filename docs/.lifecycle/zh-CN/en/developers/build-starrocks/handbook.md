---
displayed_sidebar: "Chinese"
---

# 如何构建 StarRocks

一般而言，您只需执行以下命令即可构建 StarRocks

```
./build.sh
```

此命令首先会检查所有第三方依赖项是否已准备就绪。如果所有依赖项已就绪，它将构建 StarRocks `Backend` 和 `Frontend`。

成功执行此命令后，生成的二进制文件将位于 `output` 目录中。

## 分别构建 FE/BE

您无需每次都构建 FE 和 BE，可以分别构建它们。
例如，您可以仅构建 BE，执行以下命令
```
./build.sh --be
```

然后仅构建 FE，执行以下命令
```
./build.sh --fe
```

# 如何运行单元测试

BE 和 FE 的单元测试是分开的。一般而言，您可以通过以下方式运行 BE 测试
```
./run-be-ut.sh
```

通过以下方式运行 FE 测试
```
./run-fe-ut.sh
```

## 如何在命令行中运行 BE UT

现在，BE UT 需要运行一些依赖项，`./run-be-ut.sh` 可以帮助它。但它并不够灵活。当您想要在命令行中运行 UT 时，您可以执行

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks Backend UT 基于 google-test 构建，因此您可以传递过滤器来运行其中一些 UT。例如，您只想测试与 MapColumn 相关的测试，您可以执行

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```


# 构建选项

## 用 clang 构建

您也可以使用 `clang` 构建 StarRocks

```
CC=clang CXX=clang++ ./build.sh --be
```

然后您可以在构建消息中看到类似以下的消息

```
-- compiler Clang version 14.0.0
```

## 使用不同的链接器构建

默认的链接器速度较慢，开发人员可以指定不同的链接器以加快链接速度。
例如，您可以使用 `lld`，基于 LLVM 的链接器。

您首先需要安装 `lld`。

```
sudo apt install lld
```

然后您可以使用 STARROCKS_LINKER 环境变量来设置您想要使用的链接器。
例如：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 使用不同的构建类型

您可以使用不同的 BUILD_TYPE 变量以不同类型构建 StarRocks，默认 BUILD_TYPE 是 `RELEASE`。例如，您可以通过以下方式使用 `ASAN` 类型构建 StarRocks
```
BUILD_TYPE=ASAN ./build.sh --be
```
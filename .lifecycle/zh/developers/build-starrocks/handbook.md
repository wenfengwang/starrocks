---
displayed_sidebar: English
---

# 如何构建 StarRocks

通常情况下，你可以通过执行以下命令来构建 StarRocks：

```
./build.sh
```

此命令首先会检查所有第三方依赖是否就绪。如果所有依赖项都已准备好，它将会构建 StarRocks 的后端（Backend）和前端（Frontend）。

命令成功执行后，生成的二进制文件将会在输出目录中。

## 分别构建 FE/BE

你无需每次都同时构建 FE 和 BE，可以分别独立构建。例如，你可以单独构建 BE 通过：
```
./build.sh --be
```

同样，可以只构建 FE 通过：
```
./build.sh --fe
```

# 如何运行单元测试

BE 和 FE 的单元测试是分开的。通常，你可以通过以下命令运行 BE 测试：
```
./run-be-ut.sh
```

通过以下命令运行 FE 测试：
```
./run-fe-ut.sh
```

## 如何在命令行中运行 BE 单元测试

目前，BE 的单元测试（UT）需要一些依赖才能运行，而 ./run-be-ut.sh 脚本可以帮助解决。但这种方式不够灵活。当你想在命令行中直接运行 UT 时，可以执行：

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks 后端的 UT 是基于 google-test 构建的，因此你可以传递过滤器来运行某些 UT，例如，如果你只想测试与 MapColumn 相关的测试，你可以执行：

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```

# 构建选项

## 使用 clang 构建

你也可以使用 clang 来构建 StarRocks：

```
CC=clang CXX=clang++ ./build.sh --be
```

然后你可以在构建信息中看到以下类似的消息：

```
-- compiler Clang version 14.0.0
```

## 使用不同的链接器构建

默认的链接器速度较慢，开发者可以指定不同的链接器来加快链接速度。例如，你可以使用 lld，这是基于 LLVM 的链接器。

你需要先安装 lld。

```
sudo apt install lld
```

然后你需要设置环境变量 STARROCKS_LINKER 为你想使用的链接器。例如：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 构建不同类型

你可以使用不同的 BUILD_TYPE 变量来构建不同类型的 StarRocks， 默认的 BUILD_TYPE 是 RELEASE。例如，你可以使用 ASAN 类型来构建 StarRocks，通过：
```
BUILD_TYPE=ASAN ./build.sh --be
```

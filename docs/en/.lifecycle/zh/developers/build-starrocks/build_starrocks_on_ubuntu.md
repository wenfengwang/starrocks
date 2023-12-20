---
displayed_sidebar: English
---

支持在 x86_64 和 aarch64 上构建

### 先决条件

```
sudo apt-get update
```

```
sudo apt-get install automake binutils-dev bison byacc ccache flex libiberty-dev libtool maven zip python3 python-is-python3 -y
```

### 编译器

如果 Ubuntu 版本 >= 22.04，您可以执行
```
sudo apt-get install cmake gcc g++ default-jdk -y
```

如果 Ubuntu 版本 < 22.04，请检查以下工具和编译器的版本

##### 1. GCC/G++

GCC/G++ 版本必须 >= 10.3
```
gcc --version
g++ --version
```
安装 GCC/G++（[https://gcc.gnu.org/releases.html](https://gcc.gnu.org/releases.html)）

##### 2. JDK

OpenJDK 版本必须 >= 8
```
java --version
```
安装 OpenJDK（[https://openjdk.org/install](https://openjdk.org/install)）

##### 3. CMake

CMake 版本必须 >= 3.20.1

```
cmake --version
```
安装 CMake（[https://cmake.org/download](https://cmake.org/download)）

### 提高编译速度

默认的编译并行度等于 **CPU Cores / 4**。
如果您想要提高编译速度，可以增加并行度。

1. 假设您有 32 个 CPU 核心，默认的并行度是 8。

```
./build.sh
```

2. 假设您有 32 个 CPU 核心，想使用 24 个核心来编译。

```
./build.sh -j 24
```

### 常见问题解答

1. 在 Ubuntu 20.04 中构建 `aws_cpp_sdk` 失败。
```
Error: undefined reference to pthread_create
```
该错误是由于 CMake 版本过低引起的；您可以将 CMake 版本升级到至少 3.20.1
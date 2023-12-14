---
displayed_sidebar: "English"
---

支持在x86_64和aarch64上进行构建

### 先决条件

```
sudo apt-get update
```

```
sudo apt-get install automake binutils-dev bison byacc ccache flex libiberty-dev libtool maven zip python3 python-is-python3 -y
```

### 编译器

如果Ubuntu版本 >= 22.04，你可以执行以下命令
```
sudo apt-get install cmake gcc g++ default-jdk -y
```

如果Ubuntu版本 < 22.04。
检查以下工具和编译器的版本

##### 1. GCC/G++

GCC/G++版本必须 >= 10.3
```
gcc --version
g++ --version
```
安装GCC/G++(https://gcc.gnu.org/releases.html)

##### 2. JDK

OpenJDK版本必须 >= 8
```
java --version
```
安装OpenJdk(https://openjdk.org/install)

##### 3. CMake

cmake版本必须 >= 3.20.1

```
cmake --version
```
安装cmake(https://cmake.org/download)


### 提高编译速度

默认的编译并行度等于**CPU核心数 / 4**。
如果你想提高编译速度。如果可以提高并行度。

1. 假设你有32个CPU核心，默认的并行度为8。

```
./build.sh
```

2. 假设你有32个CPU核心，想使用24个核心进行编译。

```
./build.sh -j 24
```

### 常见问题

1. 在Ubuntu 20.04上无法构建`aws_cpp_sdk`。
```
Error: undefined reference to pthread_create
```
错误来自较低的CMake版本；你可以将CMake版本升级至至少3.20.1
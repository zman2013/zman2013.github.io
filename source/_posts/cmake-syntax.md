---
title: cmake syntax
date: 2023-09-27 19:55:53
tags:
---

CMake 不是构建系统，它为其它构建工具生成构建脚本。例如：CMake 可以生成标准的 UNIX makefiles、Nigja build.ninja、Xcode project files。当然，CMake 也可以调用具体的构建工具完成构建。

# CMake Command
cmake 命令分为两步：

```shell
1. 生成构建文件
cmake -S . -B build [-G Ninja]
# -S 指定编译源码目录
# -B 指定构建文件的输出目录
# -G 指定构建系统名称: "Unix Makefiles", Ninja, 可以通过 cmake -h 查看

2. 进行构建
cmake --build build -j [--target {target}]
# 这一步也可以直接调用具体的构建工具来完成，例如 make
# --build 指定包含构建文件的目录
# -j 指定编译的线程数，如果不指定个数，会使用尽可能多的线程数，与核心数有关
# --target 指定要构建的特定目标，而不是全部
```

# CMakeLists.txt 文件内容
## 1. 设置CMake最低版本要求
`cmake_minimum_required(VERSION 3.20.0) `

## 2. 设置项目名称和语言
`project(FlatBuffers_Demo CXX)  `

## 3. 添加子目录
`add_subdirectory(examples)  `

## 4. 包含头文件目录
`include_directories(dir1/include dir2/include)`

## 5. 使用三种方式生成三个可执行文件
```shell
# 添加第一个可执行文件
add_executable(program1 program1.cpp)

# 添加第二个可执行文件（指定具体的源文件）
set(SOURCES main.cpp dir1/file1.cpp dir2/file2.cpp)
add_executable(program2 ${SOURCES})

# 添加第三个可执行文件（指定某目录下的所有源文件）
file(GLOB_RECURSE SOURCES "main.cpp" "dir1/*.cpp" "dir2/*.cpp")
add_executable(program3 ${SOURCES})
```
## 6. 内置变量
### 6.1 PROJECT_SOURCE_DIR 源文件根目录
`include_directories(${PROJECT_SOURCE_DIR}/include)`
### 6.2 PROJECT_BINARY_DIR 构建文件根目录
`include_directories(${PROJECT_BINARY_DIR}/include)`

# 其它
## 1. xxx.cmake 与 CMakeLists.txt 的关系和区别？
a. 每个目录都可以包含 CMakeLists.txt，是项目的主要构建描述文件

b. xxx.cmake 前缀可以任意命名，并且多处复用；CMakeLists.txt 命名唯一，且只服务于当前目录

c. xxx.cmake 可以定义一些共用的头文件，变量等

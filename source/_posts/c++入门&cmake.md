---
title: c++入门&cmake
date: 2022-08-17 13:34
updated: 2022-08-17 13:34
categories: c++
---

## 1. hello word

创建 hello.cpp 文件
``` cpp hello.cpp
#include <iostream>
using namespace std;
int main()
{
    cout << "Hello World";
    return 0;
}
```
编译可执行文件
```
$ g++ ./hello.cpp -o hello
```

运行 
```
$ ./hello
Hello World
```

## 2. 区分输入到stdout 和 stderr
创建 hello_std.cpp 文件
``` cpp
#include <iostream>
using namespace std;

int main(int argc, char *argv[])
{
	std::cout << "Hello World" << std::endl;
    if (argc == 2)
    {
        std::cerr << "当前不支持额外参数" << std::endl;
        return 1;
    }
    return 0;
}
```
现在使用`std::cout` 来做标准输出，使用`std::cerr` 来做错误输出，注意return也是非0

运行 
```
$ ./hello_std sds > out   # 把标准输出重定向到out文件，仅保留错误输出
当前不支持额外参数
$ echo $?    # 上次执行返回值为1
1
$ cat out    # 查看标准输出文件out 
Hello World
```

## 2. cmake编译
### 2.1 创建  config.h.in 文件
```
#define VERSION @PROJECT_VERSION@
#define VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define VERSION_MINOR @PROJECT_VERSION_MINOR@
```
以`PROJECT_`开始的3个变量是cmake的内置变量,参见[cmake project](https://cmake.org/cmake/help/latest/command/project.html#options)

### 2.2 创建 CMakeLists.txt 文件
``` cmake
cmake_minimum_required(VERSION 3.10)

# 设置项目名为hello 版本为1.0
project(hello VERSION 1.0)

# 根据config.h.in 模板文件 把cmake PROJECT_VERSION 变量输入到config.h 头文件
configure_file(config.h.in config.h)
# add the executable
add_executable(hello hello.cpp)
```

### 2.3 编译
``` bash
cmake .  # 根据CMakeLists.txt 生成 Makefile 文件
make  # 根据Makefile生成可执行文件
./hello version  # 启动应用
```
# Ubuntu20.04 配置 Vscode + CMake 开发环境

## 下载vscode

可以选择通过snap(即ubunut软件商店GUI)安装，但是snap安装可能会出现无法输入中文的情况，需要重新安装，因此最好通过以下方法安装：

**1. 官网的软件包**

[官网](https://code.visualstudio.com/Download)

在终端执行以下命令：

```shell
$ sudo dpkg -i 软件包名称
```

**2. 软件源安装**

```shell
$ curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg

$ sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/
$ sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
$ sudo apt-get install apt-transport-https

$ sudo apt-get update
$ sudo apt-get install code # or code-insiders
```

## 安装CMake

**1. 可以通过命令行安装，但是可能不是最新版本**

```shell
$ sudo apt install cmake
```

**2. 官网安装**

[官网](https://cmake.org/download/)下载`.gz`源码

解压文件

```shell
$ tar -zxv -f cmake-x.xx.x.tar.gz
```

进入Cmake文件夹，执行

```shell
$ cd cmake-x.xx.x/
$ ./bootstrap
```

编译，安装与安装之后的验证

```shell
$ make
$ sudo make install

$ cmake --version
```

**以上某个步骤出错，请查阅该[博客](https://www.cnblogs.com/yanqingyang/p/12731855.html)**


## 安装插件

安装插件 `c/c++`, `c/c++ clang`, `command adapter`, `c++ intellisense` `CMake` 和 `CMake Tools`

## 配置文件

需要配置的文件有以下几个：

- `c_cpp_properties.json` 配置项目结构引入所需要的外部头文件，自动生成和更新，输入C/C++:Edit configuration

- `tasks.json`: 构建和编译运行项目，输入Task:Configure Task

- `launch.json`: 调试配置

- `CmakeList.txt` : Cmake的配置文件，包括一些编译信息，用于生成makefile

### c_cpp_properties.json配置

`Ctrl+Shift+P`打开命令选项，选择`C/C++:Edit configuration`，将会自动生成 `c_cpp_properties.json`配置文件

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",    // 包含当前工作目录
                "/usr/local/include/**",
                "/usr/include/**"           // 库include引入 
            ],
            "defines": [],
            "compilerPath": "/usr/bin/g++",
            "cStandard": "c11",
            "cppStandard": "c++14",
            "intelliSenseMode": "linux-gcc-x64",
            "configurationProvider": "ms-vscode.cmake-tools"
        }
    ],
    "version": 4
}
```

>其实`c_cpp_properties.json`不会影响编译结果，编译过程是下面的CMake配置的，这里的配置主要是为了让Vscode找到代码所在位置

### tasks.json配置

点击F5，配置Task

`Task`,顾名思义就是任务的配置，就是说我们需要在shell中执行哪些命令，下面就是cmake编译c++项目的`tasks`了

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            // 创建buid文件夹 为什么要创建文件夹呢，因为我们的Cmake构建的文件都在这
            "label": "CreatBuildDir",	// 给这个任务取一个别名，后文要用到
            "type": "shell",
            "command": "mkdir",
            "args": [
                "-p",
                "build"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ]
        },
        {
        	// 构建，Cmake，Make编译一系列命令
            "label": "MakeBuild",
            "type": "shell",
            "command": "cd ./build ;cmake ../ ;make",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "dependsOn":[
                "CreatBuildDir"  // 在创建build目录任务结束后进行
            ]
        }
    ]
}
```

### lauch.json配置

点击左边的三角符号，点击左上方的小齿轮，添加配置（C++GDB/LLDB），修改launch.json文件，这是我们调试所需要的配置文件

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) 启动", // 调试任务名称
            "type": "cppdbg", // 任务类型
            "request": "launch", // 请求配置类型,launch或者attach
            "program": "${workspaceFolder}/build/xxxx", // main为编译后的可执行文件名,在CmakeList.txt中指定
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}/build", // 工作区间，一定是可执行文件所在的文件夹才能调试
            "environment": [],
            "externalConsole": false, // 调试时是否显示控制台窗口，一般为true显示控制台
            "MIMode": "gdb", // 指定调试器
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "MakeBuild", // task.json中的label名，我们要首先编译，再进行调试
            "miDebuggerPath": "/usr/bin/gdb"
        }
    ]
}
```

### CmakeList.txt的配置

Cmake-Tool拓展会引导你新建`CmakeList.txt`，然后你就可以根据项目进行配置了，CmakeList的语法与用法不细讲，拿下面的举个例子

```python
cmake_minimum_required(VERSION 3.0.0) 	# cmake的版本
project(xxxxx)							# 你的项目名称

include_directories(./include)			# 设置包含头文件目录
aux_source_directory(./src DIR_ROOT_SRC) # 设置根目录，即main函数所在目录

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")			# 开启警告
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -lm")			# 链接数学库
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")	# 使用c++11

# set(CMAKE_CXX_FLAGS "-O2 -DNDEBUG " )					# release-O2优化
# set(CMAKE_BUILD_TYPE "release")						# release模式

set(CMAKE_CXX_FLAGS  "-O0" )							# 调试不优化
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")			# 开启调试信息
set(CMAKE_BUILD_TYPE "debug")							# debug模式

add_executable(${PROJECT_NAME} ${DIR_ROOT_SRC})			# 设置可执行文件名称与根目录
```

## 开始编译调试

按`F5`

>具体情况按照项目本身进行调整，关键是要注意各个配置文件的作用，以及他的各个参数在编译的哪一个环节，祝好

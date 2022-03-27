# 编译原理

## 1 编译流程

编译分为四大过程：

* 预处理

  完成宏替换、文件引入，以及去除空行、注释等，为下一步的编译做准备。也就是对各种预处理命令进行处理，包括头文件的包含、宏定义的扩展、条件编译的选择等。

* 编译

  将预处理后的代码编译成汇编代码。在这个阶段中，首先要检查代码的规范性、是否有语法错误等，以确定代码实际要做的工作，在检查无误后，再把代码翻译成汇编语言。

  编译程序执行时，先分析，后综合。分析，就是指词法分析、语法分析、语义分析和中间代码生成。综合，就是指代码优化和代码生成。

  大多数的编译程序直接产生机器语言的目标代码，形成可执行的目标文件，也有的是先产生汇编语言一级的符号代码文件，再调用汇编程序进行翻译和加工处理，最后产生可执行的机器语言目标文件。

* 汇编

  汇编就是把编译阶段生成的“．s”文件转成二进制目标代码，也就是机器代码（01序列）

* 链接

  链接就是将多个目标文件以及所需的库文件链接生成可执行目标文件的过程

## 2. 动&静态库

### 2.1 静态库：

静态库实际就是一些目标文件（一般以.o结尾）的集合，静态库一般以.a结尾，只用于生成可执行文件阶段。

在链接步骤中，链接器将从库文件取得所需代码，复制到生成的可执行文件中。这种库称为静态库。其特点是可执行文件中包含了库代码的一份完整拷贝，在编译过程中被载入程序中。缺点就是多次使用就会有多份冗余拷贝，并且对程序的更新、部署和发布会带来麻烦，如果静态库有更新，那么所有使用它的程序都需要重新编译、发布

### 2.2 动态库：

动态库在链接阶段没有被复制到程序中，而是在程序运行时由系统动态加载到内存中供程序调用。

系统只需载入一次动态库，不同的程序可以得到内存中相同动态库的副本，因此节省了很多内存。

### 2.3 静态库与动态库区别

**载入时刻不同** ：

* 静态库在程序编译时会链接到目标代码中，程序运行时不再需要静态库，因此体积较大。而且每次编译都需要载入静态代码，因此内存开销大。

* 动态库在程序编译时不会被链接到目标代码中，而是在程序运行时才被载入，程序运行时需要动态库存在，因此体积较小。而且系统只需载入一次动态库，不同程序可以得到内存中相同的动态库副本，因此内存开销小。

## 3. Makefile

makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要重新编译，如何进行链接等操作。makefile 就是“自动化编译”，告诉make命令如何编译和链接。

### 3.1 Makefile语法

Makefile包含了五个重要的东西：显示规则、隐晦规则、变量定义、文件指示和注释。

1. 显示规则：显示规则说明了，如何生成一个或多个目标。这是由Makefile指出要生成的文件和文件依赖的文件。
2. 隐晦规则：基于Makefile的自动推导功能
3. 变量的定义：一般是字符串
4. 文件指示：一般是在Makefile中引用另外一个makefile文件；根据某些规则指定Makefile中有效的部分；多行
5. 注释：#指示注释



### 3.2 Makefile是如何工作的

默认方式下，输入`make`命令后：

* make 会在当前目录下找名字叫“Makefile”或“makefile”的文件。

* 如果找到，它会找文件中第一个目标文件（target），并把这个target作为最终的目标文件，如前面示例中的“main”。

* 如果 main 文件不存在，或main所依赖的.o文件的修改时间要比main文件要新，那么它会执行后面所定义的命令来生成main文件。

* 如果 main 所依赖的.o文件也存在，那么make会在当前文件中找目标为.o文件的依赖性，若找到则根据规则生成.o文件。

* make 再用.o文件声明make的终极任务，也就是执行文件“main”。

### 3.3 示例

```makefile
CC = gcc  
RM = rm  
  
CFLAGS += -D _YUQIANG  
TARGETS := myapp  
all:$(TARGETS)  
  
$(TARGETS):main.c  
$(CC) $(CFLAGS) $^ -o $@  
  
clean:  
-$(RM) -f *.o  
-$(RM) -f $(TARGETS)
```



## 4. CMake

CMake是一个跨平台的构建工具，可以用简单的语句来描述所有平台的安装（编译过程）。能够输出各种各样的makefile或者project文件。CMake并不直接构建出最终的软件，而是产生其他工具的脚本（如makefile），然后再依据这个工具的构建方式使用。

CMake是一个比make更高级的编译配置工具，它可以根据不同的平台、不同的编译器，生成相应的makefile或者vcproj项目，从而达到跨平台的目的。Android Studio利用CMake生 成的是ninja。ninja是一个小型的关注速度的构建系统。我们不需要关心ninja的脚本，知道怎么配置CMake就可以了。

CMake其实是一个跨平台的支持产出各种不同的构建脚本的一个工具。

> 在Android Studio 2.2及以上，构建原生库的默认工具是CMake。



### 4.1 CMake常用命令
#### 指定 cmake 的最小版本
```cmake
cmake_minimum_required(VERSION 3.4.1)
```
这行命令是可选的，我们可以不写这句话，但在有些情况下，如果 CMakeLists.txt 文件中使用了一些高版本 cmake 特有的一些命令的时候，就需要加上这样一行，提醒用户升级到该版本之后再执行 cmake。

#### 设置项目名称
```cmake
project(demo)
```
这个命令不是强制性的，但最好都加上。它会引入两个变量 `demo_BINARY_DIR` 和 `demo_SOURCE_DIR`，同时，cmake 自动定义了两个等价的变量 `PROJECT_BINARY_DIR` 和 `PROJECT_SOURCE_DIR`。

#### 设置编译类型
```cmake
add_executable(demo demo.cpp) # 生成可执行文件
add_library(common STATIC util.cpp) # 生成静态库
add_library(common SHARED util.cpp) # 生成动态库或共享库
add_library 默认生成是静态库，通过以上命令生成文件名字，
```
在 Linux 下是：

```
demo
libcommon.a
libcommon.so
```

在 Windows 下是：

```
demo.exe
common.lib
common.dll
```

#### 指定编译包含的源文件
##### 明确指定包含哪些源文件

```cmake
add_library(demo demo.cpp test.cpp util.cpp)
```

##### 搜索所有的 cpp 文件

aux_source_directory(dir VAR) 发现一个目录下所有的源代码文件并将列表存储在一个变量中。

```cmake
aux_source_directory(. SRC_LIST) # 搜索当前目录下的所有.cpp文件
add_library(demo ${SRC_LIST})
```
##### 自定义搜索规则
```cmake
file(GLOB SRC_LIST "*.cpp" "protocol/*.cpp")
add_library(demo ${SRC_LIST})
# 或者
file(GLOB SRC_LIST "*.cpp")
file(GLOB SRC_PROTOCOL_LIST "protocol/*.cpp")
add_library(demo ${SRC_LIST} ${SRC_PROTOCOL_LIST})
# 或者
aux_source_directory(. SRC_LIST)
aux_source_directory(protocol SRC_PROTOCOL_LIST)
add_library(demo ${SRC_LIST} ${SRC_PROTOCOL_LIST})
```

#### 查找指定的库文件
```cmake
find_library(VAR name path)查找到指定的预编译库，并将它的路径存储在变量中。
默认的搜索路径为 cmake 包含的系统库，因此如果是 NDK 的公共库只需要指定库的 name 即可。

find_library( # Sets the name of the path variable.
              log-lib
 
              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

```
类似的命令还有 find_file()、find_path()、find_program()、find_package()。

#### 设置包含的目录
```cmake
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)
```
Linux 下还可以通过如下方式设置包含的目录
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -I${CMAKE_CURRENT_SOURCE_DIR}")
```
####  设置链接库搜索目录
```cmake
link_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/libs
)
```
Linux 下还可以通过如下方式设置包含的目录
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_CURRENT_SOURCE_DIR}/libs")
```
#### 设置 target 需要链接的库
```cmake
target_link_libraries( # 目标库
                       demo
 
                       # 目标库需要链接的库
                       # log-lib 是上面 find_library 指定的变量名
                       ${log-lib} )
```
在 Windows 下，系统会根据链接库目录，搜索xxx.lib 文件，Linux 下会搜索 xxx.so 或者 xxx.a 文件，如果都存在会优先链接动态库（so 后缀）。

##### 指定链接动态库或静态库
```cmake
target_link_libraries(demo libface.a) # 链接libface.a
target_link_libraries(demo libface.so) # 链接libface.so
```
##### 指定全路径
```cmake
target_link_libraries(demo ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.a)
target_link_libraries(demo ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.so)
```
##### 指定链接多个库
```cmake
target_link_libraries(demo
    ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.a
    boost_system.a
    boost_thread
    pthread)
```
#### 设置变量
######  set 直接设置变量的值
```cmake
set(SRC_LIST main.cpp test.cpp)
add_executable(demo ${SRC_LIST})
```
###### set 追加设置变量的值
```cmake
set(SRC_LIST main.cpp)
set(SRC_LIST ${SRC_LIST} test.cpp)
add_executable(demo ${SRC_LIST})
```
###### list 追加或者删除变量的值
```cmake
set(SRC_LIST main.cpp)
list(APPEND SRC_LIST test.cpp)
list(REMOVE_ITEM SRC_LIST main.cpp)
add_executable(demo ${SRC_LIST})
```
#### 条件控制
#####  if…elseif…else…endif



逻辑判断和比较：

* `if (expression)：expression` 不为空（0,N,NO,OFF,FALSE,NOTFOUND）时为真

* `if (not exp)`：与上面相反

* `if (var1 AND var2)`

* `if (var1 OR var2)`

* ` if (COMMAND cmd)`：如果 cmd 确实是命令并可调用为真

* `if (EXISTS dir) if (EXISTS file)`：如果目录或文件存在为真

* `if (file1 IS_NEWER_THAN file2)`：当 file1 比 file2 新，或 file1/file2 中有一个不存在时为真，文件名需使用全路径

* `if (IS_DIRECTORY dir)`：当 dir 是目录时为真

* `if (DEFINED var)`：如果变量被定义为真

* `if (var MATCHES regex)`：给定的变量或者字符串能够匹配正则表达式 regex 时为真，此处 var 可以用 var 名，也可以用 ${var}

* `if (string MATCHES regex)`

  

数字比较：

* `if (variable LESS number)`：LESS 小于

* `if (string LESS number)`

* `if (variable GREATER number)`：GREATER 大于

* `if (string GREATER number)`

* `if (variable EQUAL number)`：EQUAL 等于

* `if (string EQUAL number)`

  

字母表顺序比较：

* `if (variable STRLESS string)`

* `if (string STRLESS string)`

* `if (variable STRGREATER string)`

* `if (string STRGREATER string)`

* `if (variable STREQUAL string)`

* `if (string STREQUAL string)`


示例：

```cmake
if(MSVC)
    set(LINK_LIBS common)
else()
    set(boost_thread boost_log.a boost_system.a)
endif()
target_link_libraries(demo ${LINK_LIBS})
# 或者
if(UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fpermissive -g")
else()
    add_definitions(-D_SCL_SECURE_NO_WARNINGS
    D_CRT_SECURE_NO_WARNINGS
    -D_WIN32_WINNT=0x601
    -D_WINSOCK_DEPRECATED_NO_WARNINGS)
endif()
 
if(${CMAKE_BUILD_TYPE} MATCHES "debug")
    ...
else()
    ...
endif()
```

##### while…endwhile
```cmake
while(condition)
    ...
endwhile()
```
##### foreach…endforeach
```cmake
foreach(loop_var RANGE start stop [step])
    ...
endforeach(loop_var)
start 表示起始数，stop 表示终止数，step 表示步长，示例：

foreach(i RANGE 1 9 2)
    message(${i})
endforeach(i)
# 输出：13579
```
#### 打印信息
```cmake
message(${PROJECT_SOURCE_DIR})
message("build with debug mode")
message(WARNING "this is warnning message")
message(FATAL_ERROR "this build has many error") # FATAL_ERROR 会导致编译失败
```
#### 包含其它 cmake 文件
```cmake
include(./common.cmake) # 指定包含文件的全路径
include(def) # 在搜索路径中搜索def.cmake文件
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake) # 设置include的搜索路径
```


### 4.3  CMake常用变量
#### 预定义变量
* `PROJECT_SOURCE_DIR`：工程的根目录
* `PROJECT_BINARY_DIR`：运行 cmake 命令的目录，通常是 `${PROJECT_SOURCE_DIR}/build`
* `PROJECT_NAME`：返回通过 project 命令定义的项目名称
* `CMAKE_CURRENT_SOURCE_DIR`：当前处理的 CMakeLists.txt 所在的路径
* `CMAKE_CURRENT_BINARY_DIR`：target 编译目录
* `CMAKE_CURRENT_LIST_DIR`：CMakeLists.txt 的完整路径
* `CMAKE_CURRENT_LIST_LINE`：当前所在的行
* `CMAKE_MODULE_PATH`：定义自己的 cmake 模块所在的路径，`SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)`，然后可以用INCLUDE命令来调用自己的模块
* `EXECUTABLE_OUTPUT_PATH`：重新定义目标二进制可执行文件的存放位置
* `LIBRARY_OUTPUT_PATH`：重新定义目标链接库文件的存放位置

####  环境变量
```cmake
使用环境变量
$ENV{Name}

写入环境变量
 set(ENV{Name} value) # 这里没有“$”符号
```

####  系统信息

* `CMAKE_MAJOR_VERSION`：cmake 主版本号，比如 3.4.1 中的 3
* `CMAKE_MINOR_VERSION`：cmake 次版本号，比如 3.4.1 中的 4
* `CMAKE_PATCH_VERSION`：cmake 补丁等级，比如 3.4.1 中的 1
* `CMAKE_SYSTEM`：系统名称，比如 Linux-­2.6.22
* `CMAKE_SYSTEM_NAME`：不包含版本的系统名，比如 Linux
* `CMAKE_SYSTEM_VERSION`：系统版本，比如 2.6.22
* `CMAKE_SYSTEM_PROCESSOR`：处理器名称，比如 i686
* `UNIX`：在所有的类 UNIX 平台下该值为 TRUE，包括 OS X 和 cygwin
* `WIN32`：在所有的 win32 平台下该值为 TRUE，包括 cygwin

#### 主要开关选项
*  `BUILD_SHARED_LIBS`：这个开关用来控制默认的库编译方式，如果不进行设置，使用 add_library 又没有指定库类型的情况下，默认编译生成的库都是静态库。如果 `set(BUILD_SHARED_LIBS ON)` 后，默认生成的为动态库
*  `CMAKE_C_FLAGS`：设置 C 编译选项，也可以通过指令 add_definitions() 添加
*  `CMAKE_CXX_FLAGS`：设置 C++ 编译选项，也可以通过指令 add_definitions() 添加
```camke
add_definitions(-DENABLE_DEBUG -DABC) # 参数之间用空格分隔
```


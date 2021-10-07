# cmake

## what's cmake

CMake是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。他能够输出各种各样的makefile或者project文件，CMake 的组态档取名为 CMakeLists.txt。也就是在CMakeLists.txt这个文件中写cmake代码。 一句话：cmake就是将多个cpp、hpp文件组合构建为一个大工程的语言。

## basic examples

### Hello-cmake

#### Introduction

```
file tree
    ├── CMakeLists.txt
    ├── main.cpp	
```

```c++
main.cpp
#include <iostream>
using namespace std;
int main(int argc, char *argv[])
{
   cout << "Hello CMake!" << endl;
   return 0;
}
```

​	

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5) #设置CMake最小版本
project (hello_cmake) #设置工程名
add_executable(hello_cmake main.cpp) #生成可执行文件
```

#### 解析

##### 命令作用解析

```cmake
project (hello_cmake) #设置工程名
```

CMake构建包含一个项目名称，以上命令会自动生成一些变量，在使用多个项目时引用某些变量会更加容易

比如生成了：`PROJECT_NAME`这个变量

- PROJECT_NAME 是变量名
- ${PROJECT_NAME}是变量值，值为hello_camke

```cmake
add_executable(hello_cmake main.cpp) #生成可执行文件
```

add_executable()命令指定某些源文件生成可执行文件，本节例子为main.cpp

add_executable()函数的第一个参数是一个可执行文件名，第二个参数是要编译的源文件列表

##### 生成与工程同名的二进制文件

```cmake
cmake_minimum_required(VERSION 2.6)
project (hello_cmake)
add_executable(${PROJECT_NAME} main.cpp)
```

project(hello_cmake)函数执行时会生成一个变量，是`PROJECT_NAME`，`\${PROJECT_NAME}`表示`PROJECT_NAME`变量的值为hello_cmake,所以把`${PROJECT_NAME}`用在add_executable()里可以生成可执行文件名字叫hello_cmake 

##### 外部构建与内部构建

变量`CMAKE_BINARY_DIR`指向 cmake命令的根文件夹，所有二进制文件在这个文件夹里产生。

###### 外部构建——推荐

使用外部构建，可以创建一个位于文件系统上任何位置的构建文件夹。所有临时构建和目标文件都位于此目录中，以保持源代码树的整洁。

以本节例子为例

```bash
# 新建build构建文件夹，并运行cmake命令
mkdir build
cd build
cmake ..
make
./hello_cmake
输出——Hello CMake
```

产生的文件树

```
file tree
    ├── CMakeLists.txt
    ├── main.cpp
    ├── build
    │   ├── CMakeCache.txt
    │   ├── CMakeFiles
    │   │   ├── 2.8.12.2
    │   │   │   ├── CMakeCCompiler.cmake
    │   │   │   ├── CMakeCXXCompiler.cmake
    │   │   │   ├── CMakeDetermineCompilerABI_C.bin
    │   │   │   ├── CMakeDetermineCompilerABI_CXX.bin
    │   │   │   ├── CMakeSystem.cmake
    │   │   │   ├── CompilerIdC
    │   │   │   │   ├── a.out
    │   │   │   │   └── CMakeCCompilerId.c
    │   │   │   └── CompilerIdCXX
    │   │   │       ├── a.out
    │   │   │       └── CMakeCXXCompilerId.cpp
    │   │   ├── cmake.check_cache
    │   │   ├── CMakeDirectoryInformation.cmake
    │   │   ├── CMakeOutput.log
    │   │   ├── CMakeTmp
    │   │   ├── hello_cmake.dir
    │   │   │   ├── build.make
    │   │   │   ├── cmake_clean.cmake
    │   │   │   ├── DependInfo.cmake
    │   │   │   ├── depend.make
    │   │   │   ├── flags.make
    │   │   │   ├── link.txt
    │   │   │   └── progress.make
    │   │   ├── Makefile2
    │   │   ├── Makefile.cmake
    │   │   ├── progress.marks
    │   │   └── TargetDirectories.txt
    │   ├── cmake_install.cmake
    │   └── Makefile
```

可以看到，build文件夹下生成了许多二进制文件，如果要从头开始重新创建cmake环境，只需删除构建目录build，然后重新运行cmake。

###### 内部构建

内部构建将所有临时文件和源文件生成到一起，没有build文件夹，临时文件会和源代码文件混合在一起，比较混乱

### Hello-headers

#### Introduction

```
file tree
    ├── CMakeLists.txt
    ├── include
    │   └── Hello.h
    └── src
        ├── Hello.cpp
        └── main.cpp
```

```c++
Hello.h
//声明了Hello类，Hello的方法是print()
#ifndef __HELLO_H__
#define __HELLO_H__

class Hello{
public:
	void print();
};

#endif
```

```c++
Hello.cpp
//实现了Hello::print()
#include <iostream>

#include "Hello.h"
using namespace std;
void Hello::print(){
	cout<<"Hello Headers!"<<endl;
}
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)#最低cmake版本

project (Hello_headers)# project name

set(SOURCES
	src/Hello.cpp
	src/main.cpp
	)#创建一个变量，名字叫SOURCE，包含了所有的cpp文件

add_executable(hello_headers ${SOURCES})#用所有的源文件生成一个可执行文件，这里定义
#等价于add_executable(hello_headers src/Hello.cpp src/main.cpp)

target_include_dirctories(hello_headers
	PRIVATE
		${PROJECT_SOURCE_DIR}/include
		)#设置这个可执行文件hello_headers需要包含的库路径
```

```c++
main.cpp
#include "Hello.h"

int main(int agrc, char *argv[]){
        Hello hi;
        hi.print();
        return 0;
}
```



#### concepts

##### Directory Paths

CMake语法指定了许多变量，可用于帮助您在项目或源代码树中找到有用的目录。 其中一些包括：

| variable                 | info                                                         |
| ------------------------ | ------------------------------------------------------------ |
| CMAKE_SOURCE_DIR         | The root source directory                                    |
| CMAKE_CURRENT_SOURCE_DIR | The current source directory if using sub-projects and directories |
| PROJECT_SOURCE_DIR       | The source directory of the current cmake project            |
| CMAKE_BINARY_DIR         | he root binary / build directory. This is the directory where you ran the cmake command |
| CMAKE_CURRENT_BINARY_DIR | The build directory you are currently in                     |
| PROJECT_BINARY_DIR       | The build directory for the current project                  |

想仔细体会一下，可以在CMakeLists中，利用`message()`命令输出一下这些变量。

另外，这些变量不仅可以在CMakeLists中使用，同样可以在源代码.cpp中使用。

##### 源文件变量

创建一个包含源文件的变量，以便于将其轻松添加到多个命令中，例如`add_executable()`函数。

```cmake
# Create a sources variable with a link to all cpp files to compile
set(SOURCES
    src/Hello.cpp
    src/main.cpp
)

add_executable(${PROJECT_NAME} ${SOURCES})
```

在SOURCES变量中设置特定文件名的另一种方法是使用GLOB命令使用通配符模式匹配来查找文件。`file(GLOB SOURCES "src/*.cpp")`使用*这个通配符，表示所有.cpp结尾的文件都会包含到这个SOURCES变量。

对于modern CMake，***不建议对源文件使用变量。 不建议使用glob***。

相反，通常直接在`add_xxx`函数中声明源文件。 

这对于glob命令尤其重要，如果添加新的源文件，这些命令可能不会始终为您显示正确的结果。***在CMake中指定源文件的最佳方法是明确列出它们***。

##### 包含目录

当您有其他需要包含的文件夹（文件夹里有头文件）时，可以使用以下命令使编译器知道它们： `target_include_directories()`。 编译此目标时，这将使用-I标志将这些目录添加到编译器中，例如 -I /目录/路径

```cmake
target_include_directories(target
    PRIVATE #PRIVATE 标识符指定包含的范围
        ${PROJECT_SOURCE_DIR}/include
)
```

#### 构建运行详细输出

上一节的例子中，使用make命令，输出仅显示构建状态。若想查看用于调试目的的完整输出，可以在运行make时添加参数`VERBOSE=1`

```bash
mkdir build
cd build/
cmake ..
make VERBOSE=1
```

没有添加`VERBOSE=1`

```bash
#make
[ 33%] Building CXX object CMakeFiles/hello_headers.dir/src/Hello.cpp.o
[ 66%] Building CXX object CMakeFiles/hello_headers.dir/src/main.cpp.o
[100%] Linking CXX executable hello_headers
[100%] Built target hello_headers
```

添加`VERBOSE=1`

```bash
#make VERBOSE=1
/usr/bin/cmake -S/home/wenboji/cmaketest/hello_headers -B/home/wenboji/cmaketest/hello_headers/build --check-build-system CMakeFiles/Makefile.cmake 0
/usr/bin/cmake -E cmake_progress_start /home/wenboji/cmaketest/hello_headers/build/CMakeFiles /home/wenboji/cmaketest/hello_headers/build/CMakeFiles/progress.marks
make -f CMakeFiles/Makefile2 all
make[1]: Entering directory '/home/wenboji/cmaketest/hello_headers/build'
make -f CMakeFiles/hello_headers.dir/build.make CMakeFiles/hello_headers.dir/depend
make[2]: Entering directory '/home/wenboji/cmaketest/hello_headers/build'
cd /home/wenboji/cmaketest/hello_headers/build && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/wenboji/cmaketest/hello_headers /home/wenboji/cmaketest/hello_headers /home/wenboji/cmaketest/hello_headers/build /home/wenboji/cmaketest/hello_headers/build /home/wenboji/cmaketest/hello_headers/build/CMakeFiles/hello_headers.dir/DependInfo.cmake --color=
Dependee "/home/wenboji/cmaketest/hello_headers/build/CMakeFiles/hello_headers.dir/DependInfo.cmake" is newer than depender "/home/wenboji/cmaketest/hello_headers/build/CMakeFiles/hello_headers.dir/depend.internal".
Dependee "/home/wenboji/cmaketest/hello_headers/build/CMakeFiles/CMakeDirectoryInformation.cmake" is newer than depender "/home/wenboji/cmaketest/hello_headers/build/CMakeFiles/hello_headers.dir/depend.internal".
Scanning dependencies of target hello_headers
make[2]: Leaving directory '/home/wenboji/cmaketest/hello_headers/build'
make -f CMakeFiles/hello_headers.dir/build.make CMakeFiles/hello_headers.dir/build
make[2]: Entering directory '/home/wenboji/cmaketest/hello_headers/build'
[ 33%] Building CXX object CMakeFiles/hello_headers.dir/src/Hello.cpp.o
/usr/bin/c++   -I/home/wenboji/cmaketest/hello_headers/include   -o CMakeFiles/hello_headers.dir/src/Hello.cpp.o -c /home/wenboji/cmaketest/hello_headers/src/Hello.cpp
[ 66%] Building CXX object CMakeFiles/hello_headers.dir/src/main.cpp.o
/usr/bin/c++   -I/home/wenboji/cmaketest/hello_headers/include   -o CMakeFiles/hello_headers.dir/src/main.cpp.o -c /home/wenboji/cmaketest/hello_headers/src/main.cpp
[100%] Linking CXX executable hello_headers
/usr/bin/cmake -E cmake_link_script CMakeFiles/hello_headers.dir/link.txt --verbose=1
/usr/bin/c++     CMakeFiles/hello_headers.dir/src/Hello.cpp.o CMakeFiles/hello_headers.dir/src/main.cpp.o  -o hello_headers 
make[2]: Leaving directory '/home/wenboji/cmaketest/hello_headers/build'
[100%] Built target hello_headers
make[1]: Leaving directory '/home/wenboji/cmaketest/hello_headers/build'
/usr/bin/cmake -E cmake_progress_start /home/wenboji/cmaketest/hello_headers/build/CMakeFiles 0
```

### Static-library

#### Introction

```
file tree
	├── CMakeLists.txt
    ├── include
    │   └── static
    │       └── Hello.h
    └── src
        ├── Hello.cpp
        └── main.cpp
```

```c++
Hello.h
/*声明了Hello类，Hello的方法是print(),*/

#ifndef __HELLO_H__
#define __HELLO_H__

class Hello
{
public:
    void print();
};

#endif
```

```c++
Hello.cpp
/*实现了Hello::print()*/
#include <iostream>

#include "static/Hello.h"
using namespace std;
void Hello::print()
{
    cout << "Hello Static Library!" << endl;
}
```

```c++
main.cpp
#include "static/Hello.h"

int main(int argc, char *argv[])
{
    Hello hi;
    hi.print();
    return 0;
}
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)
project(hello_library)
############################################################
# Create a library
############################################################
#库的源文件Hello.cpp生成静态库hello_library
add_library(hello_library STATIC 
    src/Hello.cpp
)
target_include_directories(hello_library
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)
# target_include_directories为一个目标（可能是一个库library也可能是可执行文件）添加头文件路径。
############################################################
# Create an executable
############################################################
# Add an executable with the above sources
#指定用哪个源文件生成可执行文件
add_executable(hello_binary 
    src/main.cpp
)
#链接可执行文件和静态库
target_link_libraries( hello_binary
    PRIVATE 
        hello_library
)
#链接库和包含头文件都有关于scope这三个关键字的用法。
```

#### CMake解析

##### 创建静态库

add_library（）函数用于从某些源文件创建一个库，默认生成在构建文件夹。 写法如下：

```cmake
add_library(hello_library STATIC
    src/Hello.cpp
)
```

在add_library调用中包含了源文件，用于创建名称为libhello_library.a的静态库

***建议：将源文件直接传递给add_library调用，而不是先把Hello.cpp赋给一个变量***

##### 添加头文件所在的目录

使用target_include_directories（）添加了一个目录，这个目录是库所包含的头文件的目录，并设置库属性为PUBLIC。

```cmake
target_include_directories(hello_library
    PUBLIC
        ${PROJECT_SOURCE_DIR}/include
)
```

使用这个函数后，这个目录会在以下情况被调用：

- 编译这个库的时候

    因为这个库hello_library由Hello.cpp生成，Hello.cpp中函数的定义在Hello.h中，Hello.h在这个include目录下，所以显然编译这个库的时候，这个目录会用到

- 编译链接到这个库hello_library的任何其他目标（库或者可执行文件）

##### private public interface 的范围详解

- PRIVATE - 目录被添加到目标（库）的包含路径中。
- INTERFACE - 目录没有被添加到目标（库）的包含路径中，而是链接了这个库的其他目标（库或者可执行程序）包含路径中
- PUBLIC - 目录既被添加到目标（库）的包含路径中，同时添加到了链接了这个库的其他目标（库或者可执行程序）的包含路径中

也就是说，根据库是否包含这个路径，以及调用了这个库的其他目标是否包含这个路径，可以分为三种scope。

**建议：**

**对于公共的头文件，最好在include文件夹下建立子目录。** 

**传递给函数target_include_directories()的目录，应该是所有包含目录的根目录，然后在这个根目录下建立不同的文件夹，分别写头文件。**

**这样使用的时候，不需要写${PROJECT_SOURCE_DIR}/include，而是直接选择对应的文件夹里对应头文件。下面是例子：`#include "static/Hello.h"`而不是`#include "Hello.h"`使用此方法意味着在项目中使用多个库时，头文件名冲突的可能性较小。**

##### 链接库

创建将使用这个库的可执行文件时，必须告知编译器需要用到这个库。 可以使用target_link_library（）函数完成此操作。add_executable()连接源文件，target_link_libraries()连接库文件。

```cmake
add_executable(hello_binary
    src/main.cpp
)

target_link_libraries( hello_binary
    PRIVATE
        hello_library
)
```

这告诉CMake在链接期间将hello_library链接到hello_binary可执行文件。 同时，这个被链接的库如果有INTERFACE或者PUBLIC属性的包含目录，那么，这个包含目录也会被传递（ propagate ）给这个可执行文件。

官方解释：https://cmake.org/cmake/help/v3.0/command/target_include_directories.html

对于

```cmake
target_link_libraries( hello_binary    
				PRIVATE											        					hello_library )
```

这个命令中的scope关键字，private，public以及interface可以举例理解：

public是说，你的这个工程如果被link了，那你的target_link_libraries指定的lib也会被link

private是说，你link的这些libs不会被暴露出去。

**比如你的工程B是个dll，public连接了C, D 这个时候你的A.exe要链接B，那么它也会链接C和D 如果B是private链接了C, D 那么A链B的时候，不会链C和D**

那么，A.exe链接B的时候，其实也有public和private的选项，但是因为没有其他东西链接A，所以不起作用。 这个主要是针对其它工程链自己的设置

对于hello_binary，它不是库，所以不会被链接。直接private自己用这个库就行。

#### 构建运行

```bash
#mkdir build
#cd build 
#cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/static_library/build
#make
Scanning dependencies of target hello_library
[ 25%] Building CXX object CMakeFiles/hello_library.dir/src/Hello.cpp.o
[ 50%] Linking CXX static library libhello_library.a
[ 50%] Built target hello_library
Scanning dependencies of target hello_binary
[ 75%] Building CXX object CMakeFiles/hello_binary.dir/src/main.cpp.o
[100%] Linking CXX executable hello_binary
[100%] Built target hello_binary
#ls
CMakeCache.txt  cmake_install.cmake  libhello_library.a
CMakeFiles      hello_binary         Makefile
#./hello_binary
Hello static library
```

### Shared-library

#### Introduction

```
file tree
    ├── CMakeLists.txt
    ├── include
    │   └── shared
    │       └── Hello.h
    └── src
        ├── Hello.cpp
        └── main.cpp
```

```c++
Hello.h
/*声明了Hello类，Hello的方法是print(),*/
#ifndef __HELLO_H__
#define __HELLO_H__

class Hello
{
public:
    void print();
};

#endif
```

```c++
Hello.cpp
/*实现了Hello::print()*/
#include <iostream>

#include "shared/Hello.h"

void Hello::print()
{
    std::cout << "Hello Shared Library!" << std::endl;
}
```

```c++
main.cpp
#include "shared/Hello.h"

int main(int argc, char *argv[])
{
    Hello hi;
    hi.print();
    return 0;
}  
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)
project(hello_library)
############################################################
# Create a library
############################################################
#根据Hello.cpp生成动态库
add_library(hello_library SHARED 
    src/Hello.cpp
)
#给动态库hello_library起一个别的名字hello::library
add_library(hello::library ALIAS hello_library)
#为这个库目标，添加头文件路径，PUBLIC表示包含了这个库的目标也会包含这个路径
target_include_directories(hello_library
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)
############################################################
# Create an executable
############################################################
#根据main.cpp生成可执行文件
add_executable(hello_binary
    src/main.cpp
)
#链接库和可执行文件，使用的是这个库的别名。PRIVATE 表示
target_link_libraries( hello_binary
    PRIVATE 
        hello::library
)
```

#### CMake解析

##### 创建动态库

`add_library()`函数用于从某些源文件创建一个动态库，默认生成在构建文件夹

```cmake
add_library(hello_library
			SHARED
				src/Hello.cpp
				)
```

在add_library调用中包含了源文件，用于创建名称为`libhello_library.so`的动态库

注：静态库后缀为`libhello_library.a`

##### 创建别名目标

别名目标是在只读上下文中可以代替真实目标名称的替代名称

```cmake
add_library(hello::library 
				ALIAS 
					hello_library
					)
```

如下所示，当将目标链接到其他目标时，使用别名可以引用目标

链接共享库和链接静态库相同，创建可执行文件时，请使用`target_link_library()`函数指向库

```cmake
add_executable(hello_binary
    src/main.cpp
)

target_link_libraries(hello_binary
    PRIVATE
        hello::library
)
```

这告诉CMake使用别名目标名称将hello_library链接到hello_binary可执行文件

#### 构建运行

```bash
# cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/shared_library/build
#make
Scanning dependencies of target hello_library
[ 25%] Building CXX object CMakeFiles/hello_library.dir/src/Hello.cpp.o
[ 50%] Linking CXX shared library libhello_library.so
[ 50%] Built target hello_library
Scanning dependencies of target hello_binary
[ 75%] Building CXX object CMakeFiles/hello_binary.dir/src/main.cpp.o
[100%] Linking CXX executable hello_binary
[100%] Built target hello_binary
#./hello_binary
Hello shared library!
```

### Installing

### Build-type

#### Introduction

```
file tree
    ├── CMakeLists.txt
    ├── main.cpp
```

```c++
main.cpp
#include <iostream>

int main(int argc, char *argv[])
{
   std::cout << "Hello Build Type!" << std::endl;
   return 0;
}
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)
#如果没有指定则设置默认编译方式
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  #在命令行中输出message里的信息
  message("Setting build type to 'RelWithDebInfo' as none was specified.")
  #不管CACHE里有没有设置过CMAKE_BUILD_TYPE这个变量，都强制赋值这个值为RelWithDebInfo
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)

  # 当使用cmake-gui的时候，设置构建级别的四个可选项
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()


project (build_type)
add_executable(cmake_examples_build_type main.cpp)

#命令的具体解释在二  CMake解析中，这里的注释只说明注释后每一句的作用
```

#### CMake解析

##### 构建级别

CMake具有许多内置的构建配置，可用于编译工程。 这些配置指定了代码优化的级别，以及调试信息是否包含在二进制文件中。 

这些优化级别，主要有：

- Release —— 不可以打断点调试，程序开发完成后发行使用的版本，占的体积小。 它对代码做了优化，因此速度会非常快， 

    在编译器中使用命令： `-O3 -DNDEBUG` 可选择此版本。

- Debug ——调试的版本，体积大。

    在编译器中使用命令： `-g` 可选择此版本。

- MinSizeRel—— 最小体积版本

    在编译器中使用命令：`-Os -DNDEBUG`可选择此版本。

- RelWithDebInfo—— 既优化又能调试。

    在编译器中使用命令：`-O2 -g -DNDEBUG`可选择此版本。

##### 设置级别的方式

###### CMake图形界面

![cmake-gui](https://gitee.com/umecjf/figures/raw/master/cmake-gui.png)

###### CMake命令行

在命令行运行CMake的时候， 使用cmake命令行的-D选项配置编译类型 

```bash
cmake .. -DCMAKE_BUILD_TYPE=Release
```

###### CMake中设置默认的构建级别

 CMake提供的默认构建类型是不进行优化的构建级别。 对于某些项目，需要自己设置默认的构建类型，以便不必记住进行设置。 

- `set()`命令

  该命令可以为普通变量、CACHE变量、环境变量赋值。

  `set()`可以设置零个或多个参数。多个参数将以[分号分隔的列表](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#cmake-language-lists)形式加入，以形成要设置的实际变量值。零参数将导致未设置普通变量。见[`unset()`](https://cmake.org/cmake/help/latest/command/unset.html#command:unset) 命令显式取消设置变量。 

  所以此处学习SET命令需要分为设置普通变量，CACHE变量以及环境变量三种类别来学习

  - 正常变量

    - ```cmake
      set(<variable> <value>... [PARENT_SCOPE])
      ```

      设置的变量值 作用域属于整个 CMakeLists.txt 文件。(一个工程可能有多个CMakeLists.txt)

      当这个语句中加入PARENT_SCOPE后，表示要设置的变量是父目录中的CMakeLists.txt设置的变量。

      比如有如下目录树：

      ```
      ├── CMakeLists.txt
      └── src
          └── CMakeLists.txt
      ```

      并且在 顶层的CMakeLists.txt中包含了src目录：`add_subdirectory(src)`

      那么，顶层的CMakeLists.txt就是父目录，

      如果父目录中有变量`Bang`,在子目录中可以直接使用（比如用message输出`Bang`，值是父目录中设置的值）并且利用set()修改该变量`Bang`的值，但是如果希望在出去该子CMakeLists.txt对该变量做出的修改能够得到保留，那么就需要在set()命令中加入Parent scope这个变量。当然，如果父目录中本身没有这个变量，子目录中仍然使用了parent scope，那么出了这个作用域后，该变量仍然不会存在。

  - CACHE变量

    - ```cmake
      set(<variable> <value>... CACHE <type> <docstring> [FORCE])
      ```

      - 首先什么是CACHE变量，就是在运行cmake的时候，变量的值可能会被缓存到一份文件里即build命令下的CMakeCache.txt，当你重新运行cmake的时候，那些变量会默认使用这个缓存里的值。这个变量是全局变量，整个CMake工程都可以使用该变量。
      - 在这个文件里，只要运行cmake ..命令，自动会出现一些值，比如 CMAKE_INSTALL_PREFIX ，如果设置  set(CMAKE_INSTALL_PREFIX "/usr") ，虽然CACHE缓存文件里还有这个CMAKE_INSTALL_PREFIX  变量，但是因为我们显示得设置了一个名为CMAKE_INSTALL_PREFIX 的正常变量，所以之后使用CMAKE_INSTALL_PREFIX ，值是我们设置的正常变量的值。
      - 如果加上CACHE关键字，则设置的这个变量会被写入缓存文件中（但如果本身缓存文件中有这个变量，则不会覆盖缓存中的变量）。只有加上FORCE关键字，这个被写入文件的值会覆盖之前文件中存在的同名变量。
      - 加上CACHE关键字，和是必需的。 

      被 CMake GUI 用来选择一个窗口，让用户设置值。可以有5种选项。其中一个是STRING  ，弹出提示消息

      - 为BOOL，则为布尔`ON/OFF`值。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供一个复选框。
      - 为FILEPATH，则为磁盘上文件的路径。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供一个文件对话框。
      - 为 PATH ，则为磁盘上目录的路径。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供一个文件对话框。
      - 为 STRING ，则为一行文字。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供文本字段或下拉选择（如果 [`STRINGS`](https://cmake.org/cmake/help/latest/prop_cache/STRINGS.html#prop_cache:STRINGS) 设置了缓存条目属性。）
      - 为 INTERNAL ，则为一行文字。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1))不显示内部条目。它们可用于在运行之间持久存储变量。使用此类型暗含`FORCE`。

      比如

      ```cmake
      set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
      ```

      这句话，就是强制在缓存文件中覆盖CMAKE_BUILD_TYPE这个变量，将这个变量设置为RelWithDebInfo。而`STRING "Choose the type of build."`参数在使用cmake-gui的时候起作用，在界面上会出现一个下拉框供给用户选择来设置CMAKE_BUILD_TYPE变量。里的一行文字作为提示。

      但是这个下拉框里的内容，需要使用随后的`set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")`这个命令来设置。也就是所谓的设置string缓存条目属性。

      ![cmake-gui](https://gitee.com/umecjf/figures/raw/master/cmake-gui.png)

      官方文档： https://cmake.org/cmake/help/latest/command/set.html 

  - 环境变量

    - ```cmake
      set(ENV{<variable>} [<value>])
      ```

      设置一个 [`Environment Variable`](https://cmake.org/cmake/help/latest/manual/cmake-env-variables.7.html#manual:cmake-env-variables(7)) 到给定值。随后的调用`$ENV{<varible>}`将返回此新值。

      此命令仅影响当前的CMake进程，不影响调用CMake的进程，也不影响整个系统环境，也不影响后续构建或测试过程的环境。

      如果在空字符串之后`ENV{}`或如果没有参数，则此命令将清除环境变量的任何现有值。之后的参数将被忽略。如果发现其他参数，则会发出警告。

#### 构建运行

```bash
#cmake ..
setting build type to 'ReiWithDebInfo' as none was specified
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/build-type/build
#make VERBOSE=1
/usr/bin/cmake -S/home/wenboji/cmaketest/build-type -B/home/wenboji/cmaketest/build-type/build --check-build-system CMakeFiles/Makefile.cmake 0
/usr/bin/cmake -E cmake_progress_start /home/wenboji/cmaketest/build-type/build/CMakeFiles /home/wenboji/cmaketest/build-type/build/CMakeFiles/progress.marks
make -f CMakeFiles/Makefile2 all
make[1]: Entering directory '/home/wenboji/cmaketest/build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/depend
make[2]: Entering directory '/home/wenboji/cmaketest/build-type/build'
cd /home/wenboji/cmaketest/build-type/build && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/wenboji/cmaketest/build-type /home/wenboji/cmaketest/build-type /home/wenboji/cmaketest/build-type/build /home/wenboji/cmaketest/build-type/build /home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake --color=
Dependee "/home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake" is newer than depender "/home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Dependee "/home/wenboji/cmaketest/build-type/build/CMakeFiles/CMakeDirectoryInformation.cmake" is newer than depender "/home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Scanning dependencies of target cmake_examples_build_type
make[2]: Leaving directory '/home/wenboji/cmaketest/build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/build
make[2]: Entering directory '/home/wenboji/cmaketest/build-type/build'
[ 50%] Building CXX object CMakeFiles/cmake_examples_build_type.dir/main.cpp.o
/usr/bin/c++     -o CMakeFiles/cmake_examples_build_type.dir/main.cpp.o -c /home/wenboji/cmaketest/build-type/main.cpp
[100%] Linking CXX executable cmake_examples_build_type
/usr/bin/cmake -E cmake_link_script CMakeFiles/cmake_examples_build_type.dir/link.txt --verbose=1
/usr/bin/c++     CMakeFiles/cmake_examples_build_type.dir/main.cpp.o  -o cmake_examples_build_type 
make[2]: Leaving directory '/home/wenboji/cmaketest/build-type/build'
[100%] Built target cmake_examples_build_type
make[1]: Leaving directory '/home/wenboji/cmaketest/build-type/build'
/usr/bin/cmake -E cmake_progress_start /home/wenboji/cmaketest/build-type/build/CMakeFiles 0


#指定构建级别
#cmake .. -DCMAKE_BUILD_TYPE=Release
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/build-type/build
#make VERBOSE=1
/usr/bin/cmake -S/home/wenboji/cmaketest/build-type -B/home/wenboji/cmaketest/build-type/build --check-build-system CMakeFiles/Makefile.cmake 0
/usr/bin/cmake -E cmake_progress_start /home/wenboji/cmaketest/build-type/build/CMakeFiles /home/wenboji/cmaketest/build-type/build/CMakeFiles/progress.marks
make -f CMakeFiles/Makefile2 all
make[1]: Entering directory '/home/wenboji/cmaketest/build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/depend
make[2]: Entering directory '/home/wenboji/cmaketest/build-type/build'
cd /home/wenboji/cmaketest/build-type/build && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/wenboji/cmaketest/build-type /home/wenboji/cmaketest/build-type /home/wenboji/cmaketest/build-type/build /home/wenboji/cmaketest/build-type/build /home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake --color=
Dependee "/home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake" is newer than depender "/home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Dependee "/home/wenboji/cmaketest/build-type/build/CMakeFiles/CMakeDirectoryInformation.cmake" is newer than depender "/home/wenboji/cmaketest/build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Scanning dependencies of target cmake_examples_build_type
make[2]: Leaving directory '/home/wenboji/cmaketest/build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/build
make[2]: Entering directory '/home/wenboji/cmaketest/build-type/build'
[ 50%] Building CXX object CMakeFiles/cmake_examples_build_type.dir/main.cpp.o
/usr/bin/c++    -O3 -DNDEBUG   -o CMakeFiles/cmake_examples_build_type.dir/main.cpp.o -c /home/wenboji/cmaketest/build-type/main.cpp
[100%] Linking CXX executable cmake_examples_build_type
/usr/bin/cmake -E cmake_link_script CMakeFiles/cmake_examples_build_type.dir/link.txt --verbose=1
/usr/bin/c++  -O3 -DNDEBUG   CMakeFiles/cmake_examples_build_type.dir/main.cpp.o  -o cmake_examples_build_type 
make[2]: Leaving directory '/home/wenboji/cmaketest/build-type/build'
[100%] Built target cmake_examples_build_type
make[1]: Leaving directory '/home/wenboji/cmaketest/build-type/build'
/usr/bin/cmake -E cmake_progress_start /home/wenboji/cmaketest/build-type/build/CMakeFiles 0
```

### Complie-flags

#### Introduction

```
file tree
├── CMakeLists.txt
├── main.cpp
```

```c++
main.cpp
#include <iostream>

int main(int argc, char *argv[])
{
   std::cout << "Hello Compile Flags!" << std::endl;

   // only print if compile flag set
#ifdef EX2
  std::cout << "Hello Compile Flag EX2!" << std::endl;
#endif

#ifdef EX3
  std::cout << "Hello Compile Flag EX3!" << std::endl;
#endif

   return 0;
}

```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)
#强制设置默认C++编译标志变量为缓存变量，如CMake（五） build type所说，该缓存变量被定义在文件中，相当于全局变量，源文件中也可以使用这个变量
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)

project (compile_flags)

add_executable(cmake_examples_compile_flags main.cpp)
#为可执行文件添加私有编译定义
target_compile_definitions(cmake_examples_compile_flags 
    PRIVATE EX3
)
#命令的具体解释在二  CMake解析中，这里的注释只说明注释后每一句的作用

```

#### CMake解析

##### 设置每个目标编译标志

在现代CMake中设置C ++标志的推荐方法是专门针对某个目标（target）设置标志，可以通过target_compile_definitions（）函数设置某个目标的编译标志。 

```cmake
target_compile_definitions(cmake_examples_compile_flags
    PRIVATE EX3
)
```

 如果这个目标是一个库（cmake_examples_compile_flags），编译器在编译目标时添加定义`-DEX3 ` ，并且选择了范围`PUBLIC`或`INTERFACE`，该定义`-DEX3`也将包含在链接此目标（cmake_examples_compile_flags）的所有可执行文件中。  注意，本语句使用了`PRIVATE`，所以编译选项不会传递。

对于编译器选项，还可以使用`target_compile_options()`函数。 

```cmake
target_compile_definitions(<target>
		<INTERFACE|PUBLIC|PRIVATE> [items1...]
		[<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

 是给 `target` 添加编译选项， `target` 指的是由 `add_executable()`产生的可执行文件或 `add_library()`添加进来的库。`<INTERFACE|PUBLIC|PRIVATE>`指的是`[items...]` 选项可以传播的范围， `PUBLIC and INTERFACE`会传播 `<target>`的 [INTERFACE_COMPILE_DEFINITIONS](https://cmake.org/cmake/help/v3.0/prop_tgt/INTERFACE_COMPILE_DEFINITIONS.html#prop_tgt:INTERFACE_COMPILE_DEFINITIONS) 属性， `PRIVATE and PUBLIC` 会传播 `target` 的 [COMPILE_DEFINITIONS ](https://cmake.org/cmake/help/v3.0/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS) 属性。 

##### 设置默认编译标志

默认的CMAKE_CXX_FLAGS为空或包含适用于构建类型的标志。  要设置其他默认编译标志，如下使用：

```cmake
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)
```

强制设置默认C++编译标志变量为缓存变量，如CMake（五） build type所说，该缓存变量被定义在文件中，相当于全局变量，源文件中也可以使用这个变量。这个变量原本包含的参数仍然存在，只是添加了EX2。

 `CACHE STRING "Set C++ Compiler Flags" FORCE`命令是为了强制将CMAKE_CXX_FLAGS变量 放到CMakeCache.txt文件中 

`"${CMAKE_CXX_FLAGS} -DEX2"`这个字符串可以保留原有的CMAKE_CXX_FLAGS中的参数，额外添加了一个EX2参数。注意写法：空格，并且参数前加了`-D`

类似设置CMAKE_CXX_FLAGS，还可以设置其他选项：

- 设置C编译标志： CMAKE_C_FLAGS
- 设置链接标志：CMAKE_LINKER_FLAGS.

##### 设置CMake标志

 与构建类型类似，可以使用以下方法设置全局C 编译器标志。 

1. 利用ccmake或者gui

2. 在cmake命令行中

   ```bash
    cmake .. -DCMAKE_CXX_FLAGS="-DEX3"
   ```

##### 区别

设置CMAKE_C_FLAGS和CMAKE_CXX_FLAGS将为该目录或所有包含的子目录中的所有目标全局设置一个编译器标志。 现在不建议使用该方法，首选使用target_compile_definitions函数。因此建议为每个目标设置编译标志

#### 构建运行

```bash
#cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/compile_flags/build
#make
Scanning dependencies of target cmake_examples_compile_flags
[ 50%] Building CXX object CMakeFiles/cmake_examples_compile_flags.dir/main.cpp.o
[100%] Linking CXX executable cmake_examples_compile_flags
[100%] Built target cmake_examples_compile_flags
```

### Third-party-library

```bash
#首先安装所需要的boost静态库
#先检查系统里是否已经安装
dpkg -S /usr/include/boost/version.hpp
sudo find /usr -name “libboost”
#若什么结果都没有，则进行安装，否则本小节编译会报错
sudo apt-get install libboost-all-dev
```

#### Introduction

```
file tree
├── CMakeLists.txt
├── main.cpp
```

```c++
main.cpp
#include <iostream>
#include <boost/shared_ptr.hpp>
#include <boost/filesystem.hpp>
using namespace std;
/*Boost库是为C++语言标准库提供扩展的一些C++程序库的总称，由Boost社区组织开发、
维护。Boost库可以与C++标准库完美共同工作，并且为其提供扩展功能。
*/
int main(int argc, char *argv[])
{
    cout << "Hello Third Party Include!" << endl;

    // use a shared ptr
    boost::shared_ptr<int> isp(new int(4));

    // trivial use of boost filesystem
    boost::filesystem::path path = "/usr/share/cmake/modules";
    if(path.is_relative())
    {
        cout << "Path is relative" << endl;
    }
    else
    {
        cout << "Path is not relative" << endl;
    }

   return 0;
}
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (third_party_include)
# find a boost install with the libraries filesystem and system
#使用库文件系统和系统查找boost install
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)
#这是第三方库，而不是自己生成的静态动态库
# check if boost was found
if(Boost_FOUND)
    message ("boost found")
else()
    message (FATAL_ERROR "Cannot find Boost")
endif()

# Add an executable
add_executable(third_party_include main.cpp)

# link against the boost libraries
target_link_libraries( third_party_include
    PRIVATE
        Boost::filesystem
)
```

#### CMake解析

几乎所有项目都将要求包含第三方库，头文件或程序。 CMake支持使用`find_package()`函数查找这些工具的路径。  这将从`CMAKE_MODULE_PATH`中的文件夹列表中搜索格式为“ FindXXX.cmake”的CMake模块。在linux上，默认搜索路径将是`/ usr / share / cmake / Modules`

##### finding a package

`find_package()`函数将从`CMAKE_MODULE_PATH`中的文件夹列表中搜索“`FindXXX.cmake`中的CMake模块。 `find_package`参数的确切格式取决于要查找的模块,这通常记录在`FindXXX.cmake`文件的顶部

```cmake
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)
```

参数：

- Boost-库名称。 这是用于查找模块文件FindBoost.cmake的一部分 

- 1.46.1 - 需要的boost库最低版本 

- REQUIRED - 告诉模块这是必需的，如果找不到会报错

- COMPONENTS - 要查找的库列表。从后面的参数代表的库里找boost 

 可以使用更多参数，也可以使用其他变量，后面的示例将提供更复杂的设置。 

##### exported variables

找到包后，会自动导出变量，可以通知用户在哪里可以找到库，头文件或者可执行文件。与与XXX_FOUND变量类似，它们与包绑定在一起，通常记录在`FindXXX.cmake`文件的顶部。 

本节中的对应变量

```cmake
Boost_INCLUDE_DIRS - boost头文件的路径
```

 在某些情况下，还可以通过使用ccmake或cmake-gui检查缓存来查看这些变量。 

##### Alias / Imported targets别名/导入目标

 大多数modern CMake库在其模块文件中导出别名目标。  导入目标的好处是它们也可以填充包含目录和链接的库。  例如，从CMake v3.5开始，Boost模块支持此功能。 与使用自己的别名目标相似，模块中的别名可以使引用找到的目标变得更加容易。   对于Boost，所有目标均使用Boost ::标识符，然后使用子系统名称导出。 例如，您可以使用： 

- `Boost::boost` for header only libraries

- `Boost::system` for the boost system library.

- `Boost::filesystem` for filesystem library.

  与您自己的目标一样，这些目标包括它们的依赖关系，因此与Boost :: filesystem链接将自动添加Boost :: boost和Boost :: system依赖关系。 

  要链接到导入的目标，可以使用以下命令： 

  ```cmake
  target_link_libraries( third_party_include
        PRIVATE
            Boost::filesystem
    )
  ```

#####  Non-alias targets

 尽管大多数现代库都使用导入的目标，但并非所有模块都已更新。 如果未更新库，则通常会发现以下可用变量： 

- xxx_INCLUDE_DIRS - 指向库的包含目录的变量。

- xxx_LIBRARY - 指向库路径的变量。

     然后可以将它们添加到您的target_include_directories和target_link_libraries中，如下所示： 

  ```cmake
    # Include the boost headers
    target_include_directories( third_party_include
        PRIVATE ${Boost_INCLUDE_DIRS}
    )
  
    # link against the boost libraries
    target_link_libraries( third_party_include
        PRIVATE
        ${Boost_SYSTEM_LIBRARY}
        ${Boost_FILESYSTEM_LIBRARY}
    )
  ```

#### 构建运行

```bash
#cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found Boost: /usr/lib/x86_64-linux-gnu/cmake/Boost-1.71.0/BoostConfig.cmake (found suitable version "1.71.0", minimum required is "1.46.1") found components: filesystem system 
boost found
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/third_party_library/build
#make
Scanning dependencies of target third_party_include
[ 50%] Building CXX object CMakeFiles/third_party_include.dir/main.cpp.o
[100%] Linking CXX executable third_party_include
[100%] Built target third_party_include
#./third_party_include
hello third party include
path is not relative
```

### Compiling-with-clang

本节需要先安装clang-3.6

```bash
sudo apt-get install clang-3.6 -y
```

#### Introduction

使用CMake进行构建时，可以设置C和C ++编译器。 此示例与hello-cmake示例相同，不同之处在于它显示了将编译器从默认的gcc更改为[clang](http://clang.llvm.org/)的最基本方法。

```
file tree
├── CMakeLists.txt	#包含了要运行的CMake命令
├── main.cpp		#一个简单的 "Hello World" cpp 文件
├── pre_test.sh		#删除之前配置的build文件
├── run_test.sh		#找到具体clang编译器路径，并配置cmake使用clang编译器
```

```cmake
CMakeLists.txt
# 寻找CMake的最低版本，必须大于3.5
cmake_minimum_required(VERSION 3.5)

# 设置工程名称
project (hello_cmake)

# 生成可执行文件
add_executable(hello_cmake main.cpp)
```

```c++
main.cpp
#include <iostream>

int main(int argc, char *argv[])
{
   std::cout << "Hello CMake!" << std::endl;
   return 0;
}
```

```bash
pre_test.sh


#!/bin/bash
#pre_test脚本删除之前配置的build文件，run_test运行clang，生成这次的build.clang文件
#这个脚本的作用是如果存在build.clang这个文件夹，就把它删除掉


#ROOT_DIR=`pwd`#shell脚本的语法，pwd输出文件当前所在路径,赋值给ROOT_DIR这个变量
dir="01-basic/I-compiling-with-clang"
if [ -d "$ROOT_DIR/$dir/build.clang" ]; then
    echo "deleting $dir/build.clang"
    rm -r $dir/build.clang
fi


#if then fi是shell脚本里的判断语句，如果[]里的条件为真，则执行then后面的语句
#基本格式：
#       if [判断语句]; then
#           执行语句
#       fi
#-d与路径配合，路径存在则为真
#单纯的dir等价于ls -C -b; 也就是说，默认情况下，文件在列中列出，并垂直排序，特殊字符由反斜杠转义序列表示。
#也就是说只要当前历经下存在build.clang就删除掉
#本文dir是一个变量
```

```bash
run_test.sh

#!/bin/bash 
#Ubuntu支持同时安装多个版本的clang。
#测试需要在调用cmake之前确定clang二进制文件
#这个脚本找到具体clang编译器路径，并配置cmake使用clang编译器


#相当于在shell中执行命令：whilch clang然后将返回的结果也就是路径，赋值给变量clang_bin       
clang_bin=`which clang`
clang_xx_bin=`which clang++`

if [ -z $clang_bin ]; then
    clang_ver=`dpkg --get-selections | grep clang | grep -v -m1 libclang | cut \
-f1 | cut -d '-' -f2`
    clang_bin="clang-$clang_ver"#把版本号存到变量，把版本号添加到clangC编译器和clang编译器
    clang_xx_bin="clang++-$clang_ver"
fi

echo "Will use clang [$clang_bin] and clang++ [$clang_xx_bin]"


mkdir -p build.clang && cd build.clang && \
    cmake .. -DCMAKE_C_COMPILER=$clang_bin -DCMAKE_CXX_COMPILER=$clang_xx_bin &\
& make

#which语句返回后面命令的路径
#  -z  指如果后面的路径为空则为真

#如果用which没有找到clang的二进制可执行文件，则用dpkg找到clang，并返回版本号
#dpkg –get-selections 罗列出所有包的名字并且给出了他们现在的状态比如已安装（ installed）已经卸载。（ deinstalled）
#grep clang从结果中查找到带有clang名字的
#grep -v   反转，选择不匹配的所有行。
#grep -m1  单纯的-m1表示输出1条匹配的结果之后就会停止
#grep -v -m1 libclang 输出包含clang的命令中，所有不包含libclang的一条介绍
#也就是去掉那些clang的库，找的是clang这个程序的版本。

#cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段写至标准输出。
#cut -f1  将这行按照空格？分隔之后选择第1个字段，就是clang-3.6
#cut -d '-' -f2  按照-分隔，选择第2个字段就是3.6   从而得到版本号
# ```shell
# $ dpkg --get-selections | grep clang | grep -v -m1 libclang
# clang-3.6					install
# $ dpkg --get-selections | grep clang | grep -v -m1 libclang | cut -f1
# clang-3.6
# $ dpkg --get-selections | grep clang | grep -v -m1 libclang | cut -f6
# install
# $ dpkg --get-selections | grep clang | grep -v -m1 libclang | cut -f1 | cut -d '-' -f2
# 3.6
# $ dpkg --get-selections | grep clang | grep -v -m1 libclang | cut -f1 | cut -d '-' -f1
# clang
# ```
#把每一步命令都运行一遍就知道用途了。
```

#### CMake解析

##### 编译器选项

CMake提供了控制程序编译以及链接的选项，选项如下:

- CMAKE_C_COMPILER - 用于编译c代码的程序。
- CMAKE_CXX_COMPILER - 用于编译c++代码的程序。
- CMAKE_LINKER - 用于链接二进制文件的程序。

本节提供方法为调用clang最基本的方法，之后的示例会展示更好的调用方法

##### 设置标志

就像 [Build Type](https://sfumecjf.github.io/cmake-examples-Chinese/F-build-type) 描述的那样, 可以使用cmake gui或从命令行设置CMake的参数。

此处使用命令行

```bash
cmake .. -DCMAKE_C_COMPILER=clang-3.6 -DCMAKE_CXX_COMPILER=clang++-3.6
```

注意，本文件中有`pre_test.sh``run_test.sh`两个脚本，上述操作已经写入脚本，可以用`bash`执行，指定编译器为clang并且编译工程。 上面通过设置关键字，表示运行make命令时，clang将用于编译二进制文件。 从make输出的以下几行可以看出这一点。

```bash
/usr/bin/clang++-3.6     -o CMakeFiles/hello_cmake.dir/main.cpp.o -c /home/matrim/workspace/cmake-examples/01-basic/I-compiling-with-clang/main.cpp
Linking CXX executable hello_cmake
/usr/bin/cmake -E cmake_link_script CMakeFiles/hello_cmake.dir/link.txt --verbose=1
/usr/bin/clang++-3.6       CMakeFiles/hello_cmake.dir/main.cpp.o  -o hello_cmake -rdynamic
```

#### 构建运行

```bash
#bash pre_test.sh
#bash run_test.sh
Will use clang [/usr/bin/clang] and clang++ [/usr/bin/clang++]
-- The C compiler identification is Clang 10.0.0
-- The CXX compiler identification is Clang 10.0.0
-- Check for working C compiler: /usr/bin/clang
-- Check for working C compiler: /usr/bin/clang -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/clang++
-- Check for working CXX compiler: /usr/bin/clang++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/compiling_with_clang/build.clang
#./hello_cmake
hello CMake!
```

### Building-with-ninja

CMake是一个元构建系统，可用于为许多其他构建工具创建构建文件。 本节展示了如何让CMake使用ninja构建工具

```bash
sudo apt-get install ninja-build
```

#### Introduction

```
file tree
├── CMakeLists.txt	#CMake命令
├── main.cpp		#一个简单的"Hello World" cpp文件
├── pre_test.sh
├── run_test.sh
```

```cmake
CMakeLists.txt
# Set the minimum version of CMake that can be used
# To find the cmake version run
# $ cmake --version
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (hello_cmake)

# Add an executable
add_executable(hello_cmake main.cpp)
```

```c++
main.cpp
#include <iostream>

int main(int argc, char *argv[])
{
   std::cout << "Hello CMake!" << std::endl;
   return 0;
}
```

```bash
pre_test.sh
#!/bin/bash                                                                    
ROOT_DIR=`pwd`
dir="01-basic/J-building-with-ninja"

if [ -d "$ROOT_DIR/$dir/build.ninja" ]; then
    echo "deleting $dir/build.ninja"
    rm -r $dir/build.ninja
fi
```

```bash
run_test.sh
#!/bin/bash                                                                     
# Travis-ci cmake version doesn't support ninja, so first check if it's supported                                                                              
ninja_supported=`cmake --help | grep Ninja`

if [ -z $ninja_supported ]; then
    echo "Ninja not supported"
    exit
fi

mkdir -p build.ninja && cd build.ninja &&  cmake .. -G Ninja && ninja
```

#### CMake解析

##### 生成器

CMake 负责为基础构建系统编写输入文件（例如Makefile） Running `cmake --help` will show the generators available

本人系统支持的生成器包括：

```bash
Generators

The following generators are available on this platform (* marks default):
* Unix Makefiles               = Generates standard UNIX makefiles.
  Green Hills MULTI            = Generates Green Hills MULTI files
                                 (experimental, work-in-progress).
  Ninja                        = Generates build.ninja files.
  Watcom WMake                 = Generates Watcom WMake makefiles.
  CodeBlocks - Ninja           = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles  = Generates CodeBlocks project files.
  CodeLite - Ninja             = Generates CodeLite project files.
  CodeLite - Unix Makefiles    = Generates CodeLite project files.
  Sublime Text 2 - Ninja       = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                               = Generates Sublime Text 2 project files.
  Kate - Ninja                 = Generates Kate project files.
  Kate - Unix Makefiles        = Generates Kate project files.
  Eclipse CDT4 - Ninja         = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles= Generates Eclipse CDT 4.0 project files.

```

可以看到CMake包括不同类型的生成器，例如命令行，IDE和其他生成器。

##### Command-Line Build Tool Generators

这些生成器用于命令行构建工具，例如Make和Ninja。 在使用CMake生成构建系统之前，必须先配置所选的工具链

The supported generators include:

- Borland Makefiles
- MSYS Makefiles
- MinGW Makefiles
- NMake Makefiles
- NMake Makefiles JOM
- Ninja  ninja用的
- Unix Makefiles make用的
- Watcom WMake

##### IDE Build Tool Generators

These generators are for Integrated Development Environments that  include their own compiler. Examples are Visual Studio and Xcode which  include a compiler natively.

The supported generators include:

- Visual Studio 6
- Visual Studio 7
- Visual Studio 7 .NET 2003
- Visual Studio 8 2005
- Visual Studio 9 2008
- Visual Studio 10 2010
- Visual Studio 11 2012
- Visual Studio 12 2013
- Xcode

##### Extra Generators

These are generators create a configuration to work with an  alternative IDE tool and must be included with either an IDE or  Command-Line generator.

The supported generators include:

- CodeBlocks
- CodeLite
- Eclipse CDT4
- KDevelop3
- Kate
- Sublime Text 2

##### Calling a Generator

To call a CMake generator you can use the `-G` command line switch, for example:

```bash
cmake .. -G Ninja
```

After doing the above CMake will generate the required Ninja build files, which can be run from using the `ninja` command.

```bash
$ cmake .. -G Ninja

$ ls
build.ninja  CMakeCache.txt  CMakeFiles  cmake_install.cmake  rules.ninja
```

#### 构建运行

```bash
#bash run_test.sh
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/building_with_ninja/build.ninja
#./hello_cmake
hello cmake!
```

### Imported-targets

如先前在Third-party-library中所述，较新版本的CMake允许您使用导入的ALIAS[imported](https://cmake.org/cmake/help/v3.6/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标链接第三方库

#### Introduction

```
file tree
├── CMakeLists.txt
├── main.cpp
├── run_test.sh
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (imported_targets)


# find a boost install with the libraries filesystem and system
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)

# check if boost was found
if(Boost_FOUND)
    message ("boost found")
else()
    message (FATAL_ERROR "Cannot find Boost")
endif()

# Add an executable
add_executable(imported_targets main.cpp)

# link against the boost libraries
target_link_libraries( imported_targets
    PRIVATE
        Boost::filesystem
)
```

```c++
main.cpp
#include <iostream>
#include <boost/shared_ptr.hpp>
#include <boost/filesystem.hpp>

int main(int argc, char *argv[])
{
    std::cout << "Hello Third Party Include!" << std::endl;

    // use a shared ptr                                                         
    boost::shared_ptr<int> isp(new int(4));

    // trivial use of boost filesystem                                          
    boost::filesystem::path path = "/usr/share/cmake/modules";
    if(path.is_relative())
    {
        std::cout << "Path is relative" << std::endl;
    }
    else
    {
        std::cout << "Path is not relative" << std::endl;
    }

   return 0;
}
```

```bash
run_test.sh
File Edit Options Buffers Tools Sh-Script Help                                  
#!/bin/bash                                                                     
# Make sure we have the minimum cmake version                                   
cmake_version=`cmake --version | grep version | cut -d" " -f3`

[[ "$cmake_version"  =~ ([3-9][.][5-9.][.][0-9]) ]] || exit 0

echo "correct version of cmake"
mkdir -p build && cd build && cmake .. && make
if [ $? -ne 0 ]; then
    echo "Error running example"
    exit 1
fi
```

#### CMake解析

##### 导入目标

Imported targets是FindXXX模块导出的只读目标

在CMake命令中包含boost这个库：

```cmake
target_link_libraries( imported_targets
      PRIVATE
          Boost::filesystem
  )
```

作用是自动链接Boost ::filesystem和Boost :: system库，同时还包括Boost 头文件目录

#### 构建运行

```bash
# cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found Boost: /usr/lib/x86_64-linux-gnu/cmake/Boost-1.71.0/BoostConfig.cmake (found suitable version "1.71.0", minimum required is "1.46.1") found components: filesystem system 
boost found
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/imported_targets/build
# make
Scanning dependencies of target imported_targets
[ 50%] Building CXX object CMakeFiles/imported_targets.dir/main.cpp.o
[100%] Linking CXX executable imported_targets
[100%] Built target imported_targets
#./imported_targets
Hello Third Party Include!
Path is not relative
```

### Cpp-standard

#### Introduction

自C++11和C++14发行以来，普遍做法是调用编译器以使用这些标准。 随着CMake的发展，有了新的使用这些标准的方式。

以下示例显示了设置C ++标准的不同方法，以及提供哪些版本的CMake

```
file tree
├── common-method		#可以与大多数版本的CMake一起使用的简单方法
├── cxx-standard		#使用CMake v3.1中引入的CMAKE_CXX_STANDARD变量
├── compile-features	#使用CMake v3.1中引入的target_compile_features函数
```

这节主要讲`common-method`

```
file tree
 ├── CMakeLists.txt	#Contains the CMake commands you wish to run
 ├── main.cpp		#A simple "Hello World" cpp file targeting C++11
```

```cmake
CMakeLists.txt
# Set the minimum version of CMake that can be used
# To find the cmake version run
# $ cmake --version
cmake_minimum_required(VERSION 2.8)

# Set the project name
project (hello_cpp11)

# try conditional compilation
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

# check results and add flag
if(COMPILER_SUPPORTS_CXX11)#
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)#
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# Add an executable
add_executable(hello_cpp11 main.cpp)
```

```c++
main.cpp
#include <iostream>

int main(int argc, char *argv[])
{
    auto message = "Hello C++11";
    std::cout << message << std::endl;
    return 0;
}
```

#### concepts

##### checking compile flags

CMake支持传递一个变量给函数`CMAKE_CXX_COMPILER_FLAG`来编译程序。 然后将结果存储在您传递的变量中

```cmake
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
```

这个例子将flag `-std=c++11`传递给变量`COMPILER_SUPPORTS_CXX11`

想使用这个函数，必须使用`include(CheckCXXCompilerFlag)`包含这个函数

##### adding the flag

确定编译器是否支持flag后，即可使用标准cmake方法将此标志添加到目标。 在此示例中，我们使用`CMAKE_CXX_FLAGS`将标志（c++标准）传播给所有目标

```cmake
if(COMPILER_SUPPORTS_CXX11)#
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)#
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()
```

上面的示例仅检查编译标志的gcc版本，并支持从C++11到预标准化C+\0x标志的回退。 在实际使用中，您可能需要检查C14，或添加对设置编译方法的不同支持，例如 `-std = gnu11`

##### 构建运行

```bash
$ cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Performing Test COMPILER_SUPPORTS_CXX11
-- Performing Test COMPILER_SUPPORTS_CXX11 - Success
-- Performing Test COMPILER_SUPPORTS_CXX0X
-- Performing Test COMPILER_SUPPORTS_CXX0X - Success
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/cpp_standard/i-common-method/build
$ make
Scanning dependencies of target hello_cpp11
[ 50%] Building CXX object CMakeFiles/hello_cpp11.dir/main.cpp.o
[100%] Linking CXX executable hello_cpp11
[100%] Built target hello_cpp11
$ ./hello_cpp11
Hello C++11
```

## sub project

### basic

#### Introduction

本节说明如何包含子项目，顶级CMakeLists.txt调用子目录中的CMakeLists.txt来创建以下内容：

- sublibrary1 - 一个静态库
- sublibrary2 - 只有头文件的库
- subbinary - 一个可执行文件

```
file tree
├── CMakeLists.txt		#最高层的CMakeLists.txt
├── subbinary
│   ├── CMakeLists.txt	#生成可执行文件的CMakeLists.txt
│   └── main.cpp		#可执行文件的源文件
├── sublibrary1
│   ├── CMakeLists.txt	#生成静态库的CMakeLists.txt
│   ├── include
│   │   └── sublib1
│   │       └── sublib1.h
│   └── src
│       └── sublib1.cpp
└── sublibrary2
    ├── CMakeLists.txt	#生成仅有头文件的库的CMakeLists.txt
    └── include
        └── sublib2
            └── sublib2.h
```

```cmake
CMakeLists.txt
#最高层
cmake_minimum_required (VERSION 3.5)

project(subprojects)

# Add sub directories
add_subdirectory(sublibrary1)
add_subdirectory(sublibrary2)
add_subdirectory(subbinary)
```

```cmake
subbinary/CMakeLists.txt
project(subbinary)

# Create the executable
add_executable(${PROJECT_NAME} main.cpp)

# Link the static library from subproject1 using its alias sub::lib1
# Link the header only library from subproject2 using its alias sub::lib2
# This will cause the include directories for that target to be added to this project
target_link_libraries(${PROJECT_NAME}
    sub::lib1
    sub::lib2
)
```

```c++
subbinary/main.cpp
#include "sublib1/sublib1.h"
#include "sublib2/sublib2.h"

int main(int argc, char *argv[])
{
    sublib1 hi;
    hi.print();

    sublib2 howdy;
    howdy.print();

    return 0;
}
```

#### concepts

##### 添加子目录

最高层CMakeLists.txt文件可以包含和调用包含CMakeLists.txt文件的子目录

```cmake
add_subdirectory(sublibrary1)
add_subdirectory(sublibrary2)
add_subdirectory(subbinary)
```

##### 引用子项目目录

使用`project()`命令创建项目时，CMake将自动创建许多变量，这些变量可用于引用有关该项目的详细信息。 这些变量然后可以由其他子项目或主项目使用。 例如，要引用您可以使用的其他项目的源目录

```cmake
    ${sublibrary1_SOURCE_DIR}
    ${sublibrary2_SOURCE_DIR}
```

CMake中有一些变量会自动创建:

| Variable           | Info                                                         |
| ------------------ | ------------------------------------------------------------ |
| PROJECT_NAME       | 当前project()设置的项目的名称。                              |
| CMAKE_PROJECT_NAME | 由project()命令设置的第一个项目的名称，即顶层项目。          |
| PROJECT_SOURCE_DIR | 当前项目的源文件目录。                                       |
| PROJECT_BINARY_DIR | 当前项目的构建目录。                                         |
| name_SOURCE_DIR    | 在此示例中，创建的源目录为 `sublibrary1_SOURCE_DIR`, `sublibrary2_SOURCE_DIR`, and `subbinary_SOURCE_DIR` |
| name_BINARY_DIR    | 本工程的二进制目录是`sublibrary1_BINARY_DIR`, `sublibrary2_BINARY_DIR`,和 `subbinary_BINARY_DIR` |

##### 仅含头文件的库

如果您有一个库被创建为仅头文件的库，则cmake支持INTERFACE目标，以允许创建没有任何构建输出的目标。 如果有更多问题，可以从[官方文档](https://cmake.org/cmake/help/v3.4/command/add_library.html#interface-libraries)找到更多详细信息

```cmake
add_library(${PROJECT_NAME} INTERFACE)
```

创建目标时，您还可以使用INTERFACE范围包含该目标的目录。 INTERFACE范围用于制定在链接此目标的任何库中使用的目标需求，但在目标本身的编译中不使用。

```cmake
target_include_directories(${PROJECT_NAME}
    INTERFACE
        ${PROJECT_SOURCE_DIR}/include
)
```

##### 引用子项目中的库

如果子项目创建了一个库，则其他项目可以通过在target_link_libraries()命令中调用该项目的名称来引用该库。 这意味着您不必引用新库的完整路径，而是将其添加为依赖项

```cmake
target_link_libraries(subbinary
    PUBLIC
        sublibrary1
)
```

或者，您可以创建一个别名目标，该目标允许您在上下文（其实就是某个目标的绰号）中引用该目标

```cmake
add_library(sublibrary2)
add_library(sub::lib2 ALIAS sublibrary2)
```

要想引用这些别名，可以像下面一样

```cmake
target_link_libraries(subbinary
    sub::lib2
)
```

##### 包含子项目中的目录

从cmake v3开始从子项目添加库时，无需将项目include目录添加到二进制文件的include目录中。

创建库时，这由target_include_directories()命令中的作用域控制。  在此示例中，因为子二进制可执行文件链接了sublibrary1和sublibrary2库，所以当它们与库的PUBLIC和INTERFACE范围一起导出时，它将自动包含`${sublibrary1_SOURCE_DIR} / inc`和`$ {sublibrary2_SOURCE_DIR} /  inc`文件夹。（这个地方设及到了`PUBLIC`和`INTERFACE`的使用，可以参考静态库处的讲解）

#### 构建运行

```bash
$ cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/sub_project/build
$ make
Scanning dependencies of target sublibrary1
[ 25%] Building CXX object sublibrary1/CMakeFiles/sublibrary1.dir/src/sublib1.cpp.o
[ 50%] Linking CXX static library libsublibrary1.a
[ 50%] Built target sublibrary1
Scanning dependencies of target subbinary
[ 75%] Building CXX object subbinary/CMakeFiles/subbinary.dir/main.cpp.o
[100%] Linking CXX executable subbinary
[100%] Built target subbinary
```

## code generation

### configure files generation

### Introduction

在CMake的调用过程中可能会使用CMakeList.txt和CMake缓存中的变量来创建文件；在CMake的生成过程中，该文件会被复制到新的位置，其中所有的变量将被使用他们的值替换

```
file tree
├── CMakeLists.txt	#描述你希望能够运行的CMake命令
├── main.cpp		#包含主函数的源文件
├── path.h.in		#描述待构建目录的文件
├── ver.h.in		#描述工程版本信息的文件
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (cf_example)

# set a project version
set (cf_example_VERSION_MAJOR 0)
set (cf_example_VERSION_MINOR 2)
set (cf_example_VERSION_PATCH 1)
set (cf_example_VERSION "${cf_example_VERSION_MAJOR}.${cf_example_VERSION_MINOR}.${cf_example_VERSION_PATCH}")

# Call configure files on ver.h.in to set the version.
# Uses the standard ${VARIABLE} syntax in the file
configure_file(ver.h.in ${PROJECT_BINARY_DIR}/ver.h)

# configure the path.h.in file.
# This file can only use the @VARIABLE@ syntax in the file
configure_file(path.h.in ${PROJECT_BINARY_DIR}/path.h @ONLY)

# Add an executable
add_executable(cf_example
    main.cpp
)

# include the directory with the new files
target_include_directories( cf_example
    PUBLIC
        ${CMAKE_BINARY_DIR}
)
```

```c++
main.cpp
#include <iostream>
#include "ver.h"
#include "path.h"

int main(int argc, char *argv[])
{
    std::cout << "Hello Version " << ver << "!" << std::endl;
    std::cout << "Path is " << path << std::endl;
   return 0;
}
```

```c++
path.h.in
#ifndef __PATH_H__
#define __PATH_H__

// version variable that will be substituted by cmake
// This shows an example using the @ variable type
const char* path = "@CMAKE_SOURCE_DIR@";

#endif
```

```c++
ver.h.in
#ifndef __VER_H__
#define __VER_H__

// version variable that will be substituted by cmake
// This shows an example using the $ variable type
const char* ver = "${cf_example_VERSION}";

#endif
```

### concepts

#### configure file

在CMake中，你可以在文件中使用 `configure_file()` 函数进行变量的替换，这一函数的必要参数是源文件和目标文件

```cmake
configure_file(ver.h.in ${PROJECT_BINARY_DIR}/ver.h)
configure_file(path.h.in ${PROJECT_BINARY_DIR}/path.h @ONLY)
```

第一个例子，在ver.h.in文件中，CMake可以将使用 `${}` 或 `@@` 的语法来定义一个CMake变量。在执行代码生成之后，在 `PROJECT_BINARY_DIR` 目录下将会出现一个新的ver.h文件

```c++
const char* ver = "${cf_example_VERSION}";
```

第二个例子，在path.h.in文件中，`@ONLY` 指定了它只能用 `@@` 的语法来定义一个CMake变量。同样地，在执行代码生成之后，在 `PROJECT_BINARY_DIR` 目录下将会出现一个新的path.h文件

```c++
const char* path = "@CMAKE_SOURCE_DIR@";
```

### 构建运行

```bash
$ cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/code_generation/configure-files/build
$ ls
CMakeCache.txt  CMakeFiles  cmake_install.cmake  Makefile  path.h  ver.h
cat path.h
#ifndef __PATH_H__
#define __PATH_H__

// version variable that will be substituted by cmake
// This shows an example using the @ variable type
const char* path = "/home/wenboji/cmaketest/code_generation/configure-files";

#endif

$ cat ver.h
#ifndef __VER_H__
#define __VER_H__

// version variable that will be substituted by cmake
// This shows an example using the $ variable type
const char* ver = "0.2.1";

#endif

$ make
Scanning dependencies of target cf_example
[ 50%] Building CXX object CMakeFiles/cf_example.dir/main.cpp.o
[100%] Linking CXX executable cf_example
[100%] Built target cf_example
$ ./cf_example
Hello Version 0.2.1!
Path is /home/wenboji/cmaketest/code_generation/configure-files
```

### protobuf

#### Introduction

本节展示如何使用protobuf生成源码文件

```bash
sudo apt-get install protobuf-compiler libprotobuf-dev
```

Protocol Buffers 是一种由谷歌提出的数据序列化的格式。用户可以提供一个描述了数据的 `.proto` 格式的文件，通过protobuf编译器，这一文件可以被编译成包括C++在内的一系列编程语言的源码文件

```
file tree
├── AddressBook.proto	#proto file from main protocol buffer
├── CMakeLists.txt		#Contains the CMake commands you wish to run
├── main.cpp			#The source file from the protobuf example
```

```protobuf
AddressBook.proto
package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}

message AddressBook {
  repeated Person person = 1;
}
```

```cmake
CMakeLists.txt
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (protobuf_example)

# find the protobuf compiler and libraries
find_package(Protobuf REQUIRED)

# check if protobuf was found
if(PROTOBUF_FOUND)
    message ("protobuf found")
else()
    message (FATAL_ERROR "Cannot find Protobuf")
endif()

# Generate the .h and .cxx files
PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS AddressBook.proto)

# Print path to generated files
message ("PROTO_SRCS = ${PROTO_SRCS}")
message ("PROTO_HDRS = ${PROTO_HDRS}")

# Add an executable
add_executable(protobuf_example
    main.cpp
    ${PROTO_SRCS}
    ${PROTO_HDRS})

target_include_directories(protobuf_example
    PUBLIC
    ${PROTOBUF_INCLUDE_DIRS}
    ${CMAKE_CURRENT_BINARY_DIR}
)

# link the exe against the libraries
target_link_libraries(protobuf_example
    PUBLIC
    ${PROTOBUF_LIBRARIES}
)
```

```c++
main.cpp
#include <iostream>
#include <fstream>
#include <string>
#include "AddressBook.pb.h"
using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
  cout << "Enter person ID number: ";
  int id;
  cin >> id;
  person->set_id(id);
  cin.ignore(256, '\n');

  cout << "Enter name: ";
  getline(cin, *person->mutable_name());

  cout << "Enter email address (blank for none): ";
  string email;
  getline(cin, email);
  if (!email.empty()) {
    person->set_email(email);
  }

  while (true) {
    cout << "Enter a phone number (or leave blank to finish): ";
    string number;
    getline(cin, number);
    if (number.empty()) {
      break;
    }

    tutorial::Person::PhoneNumber* phone_number = person->add_phone();
    phone_number->set_number(number);

    cout << "Is this a mobile, home, or work phone? ";
    string type;
    getline(cin, type);
    if (type == "mobile") {
      phone_number->set_type(tutorial::Person::MOBILE);
    } else if (type == "home") {
      phone_number->set_type(tutorial::Person::HOME);
    } else if (type == "work") {
      phone_number->set_type(tutorial::Person::WORK);
    } else {
      cout << "Unknown phone type.  Using default." << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!input) {
      cout << argv[1] << ": File not found.  Creating a new file." << endl;
    } else if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  // Add an address.
  PromptForAddress(address_book.add_person());

  {
    // Write the new address book back to disk.
    fstream output(argv[1], ios::out | ios::trunc | ios::binary);
    if (!address_book.SerializeToOstream(&output)) {
      cerr << "Failed to write address book." << endl;
      return -1;
    }
  }

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

#### CMake解析

##### Exported Variables

本节通过CMake protobuf包导出并使用的变量如下

- `PROTOBUF_FOUND` - 标志Protocol Buffers是否安装
- `PROTOBUF_INCLUDE_DIRS` - 描述protobuf头文件路径
- `PROTOBUF_LIBRARIES` - 描述protobuf库文件路径

关于Protobuf变量更详细的描述可以查阅 `FindProtobuf.cmake` 文件

##### Generating Source

CMake protobuf包中包含了一些列的帮助函数来简化源代码生成流程，在本示例中，我们是用下面的语句来生成C++源码：

```cmake
PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS AddressBook.proto)
```

这里的参数包括：

- PROTO_SRCS - 将被存储在.pb.cc文件里的变量名
- PROTO_HDRS- 将被存储在.pb.h文件里的变量名
- AddressBook.proto - 用于编译的原始.proto文件

##### Generated Files

在调用过 `PROTOBUF_GENERATE_CPP` 函数之后，上述变量便可以被使用了，他们将被视为protobuf编译器执行特定命令生成的输出结果

为了进一步完成文件的生成，你需要将他们添加进库函数或者可执行文件中，例如

```cmake
add_executable(protobuf_example
    main.cpp
    ${PROTO_SRCS}
    ${PROTO_HDRS})
```

这使得 `make` 这一命令被调用时，protobuf编译器也将随之被调用

当.proto文件被改变时，与其相关联的源代码文件也将被自动重新生成；不过，如果.proto文件没有发生修改，重新执行 `make` 命令，并不会发生任何变化

### 构建运行

```bash
$ cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found Protobuf: /usr/lib/x86_64-linux-gnu/libprotobuf.so;-lpthread (found version "3.6.1") 
protobuf found
PROTO_SRCS = /home/wenboji/cmaketest/code_generation/protobuf/build/AddressBook.pb.cc
PROTO_HDRS = /home/wenboji/cmaketest/code_generation/protobuf/build/AddressBook.pb.h
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wenboji/cmaketest/code_generation/protobuf/build
$ ls
CMakeCache.txt  CMakeFiles  cmake_i
nstall.cmake  Makefile
$ make
Scanning dependencies of target protobuf_example
[ 50%] Building CXX object CMakeFiles/protobuf_example.dir/main.cpp.o
[ 75%] Building CXX object CMakeFiles/protobuf_example.dir/AddressBook.pb.cc.o
[100%] Linking CXX executable protobuf_example
[100%] Built target protobuf_example
```

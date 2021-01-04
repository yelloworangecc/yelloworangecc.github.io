---
layout: post
title:  "CMake教程"
summary: CMake是一个跨平台的编译构建工具,我接触过的开源库OpenCV,QT,google test都可以使用CMake来管理代码和构建二进制文件.本文是对CMake官方教程文档的翻译.
featured-img: sleek
# categories: jekyll update
---
## 目录 ##
* CMake教程
    + 简介
    + 开始
        - 增加一个版本号和配置头文件
        - 指定C++标准
        - 构建和测试
    + 增加一个库
    + 增加库的使用要求
    + 安装和测试
        - 安装规则
        - 测试支持
    + 增加系统内省
        - 指定编译宏
    + 增加自定义命令和生成文件
    + 构建一个安装器
    + 增加对仪表盘的支持
    + 静态和动态混合
    + 增加生成表达式
    + 增加导出配置
    + 调试版本和发布版本打包
## 简介 ##
这篇CMake教程提供了对于解决通用构建系统问题的循序渐进的指导.文章将不同的主题在一个示例项目中得到呈现,对于读者理解非常有帮助.教程文档和代码示例位于CMake源码树下的Help/guide/tutorial文件夹中.教程的每一个步骤都有一个子目录,包含了相关代码.教程中的示例是渐进的,后一个步骤完全包含了前一个步骤的内容.

## 步骤一 开始 ##
最基本的项目是从源代码编译一个可执行程序.例如,一个只有3行内容的CMakeLists.txt,将是我们开始教程的地方.在Step1的文件夹内创建一个CMakeLists.txt包含下面内容:
```
cmake_minimum_required(VERSION 3.10)

# 设置项目名称
project(Tutorial)

# 添加可执行文件
add_executable(Tutorial tutorial.cxx)
```
注意,这个例子中使用了小写的命令,CMake不区分大小写.在Step1目录里有个一个tutorial.cxx源代码文件,实现了计算一个数的平方根.

### 增加一个版本号和配置头文件 ###
我们要增加的第一个特性是为可执行文件和项目指定一个版本号.尽管我们也可以在源代码中指定,使用CMakeLists.txt更加灵活.
首先,修改CMakeLists.txt中用到的project()命令,设置项目名称的同时指定版本号.
```
cmake_minimum_required(VERSION 3.10)

# 设置项目名称和版本
project(Tutorial VERSION 1.0)
```
接着,配置一个头文件,将版本号传入源代码中.
```
# 配置一个头文件,将CMake的配置传入到源代码中
configure_file(TutorialConfig.h.in TutorialConfig.h)
```
由于配置文件会被写入二进制中,我们还必须将配置文件所在目录加入到头文件搜索路径.因此需要将下面的内容加入到CMakeLists.txt文件的末尾.
```
# 为可执行文件目标添加头文件搜索路径
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```
使用你最喜欢的编辑器,在源代码目录下,创建文件TutorialConfig.h.in,文件内容如下.
```
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```
当CMake配置这个头文件时,@Tutorial_VERSION_MAJOR@和@Tutorial_VERSION_MINOR@的值会被替换.
下一步,修改tutorial.cxx以包含配置好的头文件TutorialConfig.h.
最后,修改tutorial.cxx以打印可执行文件名和版本号.
```
  if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
```
### 指定C++标准 ###
下一步,增加一些C++11新特性到我们的项目中来,在tutorial.cxx中,用std::stod替换atof,同时移除#include <cstdlib>.
```
  const double inputValue = std::stod(argv[1]);
```
我们需要使用CMake代码来明确指示正确的编译参数.最简单的,用来指定C++标准的方法是使用CMAKE_CXX_STANDARD变量.对于本教程来说,在CMakeLists.txt中设置CMAKE_CXX_STANDARD的值为11,并设置CMAKE_CXX_STANDARD_REQUIRED的值为True.确保CMAKE_CXX_STANDARD声明在调用add_executable的前面.
```
cmake_minimum_required(VERSION 3.10)

# 设置工程名称和版本
project(Tutorial VERSION 1.0)

# 指定C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```
### 构建和测试 ###
运行cmake或cmake-gui可执行程序来配置工程,随后使用指定的构建工具构建它.
例如,在命令行中导航到CMake源代码树下的Help/guide/tutorial目录中,创建一个构建目录.
```
mkdir Step1_build
```
然后,进入构建目录,运行cmake来配置工程并生成一个本地的构建系统.
```
cd Step1_build
cmake ..
```
接着,调用构建系统来真正地编译和链接工程.
```
make
```
最后,试着运行新构建的Tutorial程序.
```
Tutorial 4294967296
Tutorial 10
Tutorial
```

## 步骤二 增加一个库 ##
现在我们将向项目中增加一个库.这个库包含我们自己实现的用来计算一个数的平方根的功能.可执行程序随后可以用这个库代替编译器提供的标准库中的求平方根函数.
本教程中,我们将库放到MathFunctions子目录.这个目录中已经有一个头文件MathFunctions.h和源文件mysqrt.cxx.源文件中有一个叫mysqrt的函数,实现了编译器中sqrt函数相似的功能.
在MathFunctions子目录下增加CMakeLists.txt文件,文件包含下面的一行内容.
```
add_library(MathFunctions mysqrt.cxx)
```
为了能使用新的库,需要在顶层CMakeLists.txt增加add_subdirectory()调用,这样库才会被编译.我们向可执行文件中增加一个链接库,并且将子目录MathFunctions作为一个包含目录,这样头文件MathFunctions.h才能被找到.顶层CMakeLists.txt的最后几行内容如下.
```
# 添加MathFunctions库
add_subdirectory(MathFunctions)

# 添加可执行文件
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC MathFunctions)

# 为了能找到TutorialConfig.h,将二进制目录添加到头文件搜索路径
# 将子目录MathFunctions加入到头文件搜索路径
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```
现在,让我们将MathFunctions库配置成可选项.虽然对于本教程而言没有什么必要做额外的配置,但是对应大型项目是非常必要的.首先,是向顶层CMakeLists.txt添加一个配置.
```
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# 配置一个头文件用来向源文件传入CMake配置
configure_file(TutorialConfig.h.in TutorialConfig.h)
```
这个选项将会在cmake-gui和ccmake中以默认值ON显示,用户可以改变这个值.这个设置会被保存到缓存,用户无需每次运行CMake都去设置.
下一个变化是让CMake根据条件任去编译和链接MathFunctions库.对顶层CMakeLists.txt文件末尾的内容作如下变化.
```
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# 添加可执行文件
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# 为了能找到TutorialConfig.h,将二进制目录添加到头文件搜索路径
# 将子目录MathFunctions加入到头文件搜索路径
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ${EXTRA_INCLUDES}
                           )
```
上面例子中的新的变量EXTRA_LIBS用来收集要被链接到可执行程序的可选库,类似的,变量EXTRA_INCLUDES用来收集可选的头文件,这是一个典型的处理多个可选组件的方法,下一步我们将展示一种现代的方法.
这种方法对源代码的修改是非常直接的.首先在tutorial.cxx中包含MathFunctions.h头文件.
```
#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif
```
然后,在相同的文件中,使用USE_MYMATH控制哪一个求平方根函数将被使用.
```
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```
由于源代码使用了USE_MYMATH,需要将它添加到TutorialConfig.h.in文件中.
```
#cmakedefine USE_MYMATH
```
思考:为什么我们要在设置USE_MYMATH选项后再配置TutorialConfig.h.in?如果交换两者的顺序会发生什么?
运行cmake或cmake-gui可执行程序来配置项目,然后使用指定的工具构建,接着运行Tutorial可执行程序.
现在,让我们更新USE_MYMATH的值.最简单的方式是使用cmake-gui或ccmake(两者都是图形界面工具,前者运行在windows系统,后者运行在类unix系统,而cmake是命令行工具).或者直接通过以下命令来改变选项设置.
```
cmake .. -DUSE_MYMATH=OFF
```
重新构建和运行tutorial程序.
sqrt和mysqrt哪一个函数得到的值更准确?

## 步骤三 增加库的使用要求 ##
"使用要求"允许对库或可执行文件的链接和包含关系进行更好的控制,同时对CMake内部目标的传递属性给与更多控制.涉及的主要命令如下.
```
target_compile_definitions()
target_compile_options()
target_include_directories()
target_link_libraries()
```
让我们使用新的方式重构步骤二的代码.任何链接MathFunctions的对象都需要包含源文件目录,但是MathFunctions本身不需要.所以这种情况是一种"接口使用要求".
接口意味着客户需要的但是生产者不需要.将下面内容添加到MathFunctions/CMakeLists.txt文件的末尾.
```
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )
```
现在,我们已经为MathFunctions指定了"使用要求",于是我们可以安全地将步骤二中的EXTRA_INCLUDES变量从顶层CMakeLists.txt中移除(共2处).
```
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()
```

```
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```
完成后,在构建目录中运行cmake或cmake-gui配置项目并使用你指定的构建工具执行构建动作即可.

## 步骤四 安装和测试 ##
现在,我们可以给我们的项目添加安装规则和测试支持.

### 安装规则 ###
安装规则相当简单:我们希望为MathFunctions安装库和头文件,为应用程序安装可执行文件和配置头文件.
因此在MathFunctions/CMakeLists.txt文件末尾添加如下内容:
```
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```
在顶层CMakeLists.txt文件末尾添加如下内容
```
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```
以上就创建了一个基础本地安装了.
现在运行cmake或cmakej-gui来配置项目,然后使用你指定的构建工具执行构建动作.
接着在命令行中使用cmake的install选项执行安装步骤.对于多配置工具,不要忘记使用--config来指定一个配置.如果使用了IDE,那么构建INSTALL目标即可.这一步会安装合适的头文件,库和可执行文件.
```
cmake --install .
```
cmake的CMAKE_INSTALL_PREFIX变量用来指定文件被安装的根目录.在使用cmake --install命令时,可以使用--prefix选项覆盖变量已有的值.
```
cmake --install . --prefix "/home/myuser/installdir"
```
现在进入安装目录,检查命令是否执行成功.

### 测试支持 ###
下一步,让我们测试应用程序.在顶层CMakeLists.txt文件末尾开启测试功能,然后添加一些基础测试用例来验证程序是否正确执行.
```
enable_testing()

# 测试程序是否运行
add_test(NAME Runs COMMAND Tutorial 25)

# 测试帮助信息是否正确
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# 顶一个函数来添加测试用例
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# 执行一系列的基础测试
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```
第一个测试用例简单地测试了程序是否运行起来并且返回0.失败的情况是出现段错误或程序崩溃.这也是ctest最基础的测试用例.
下一个测试用例使用了测试属性PASS_REGULAR_EXPRESSION来验证测试的输出是否包含特定的字符串.在本例中,当参数数目不正确时,检查帮助信息是否被打印.
最后,我们创建了do_test函数来运行应用程序并测试给定的输入数值的平方根是否被正确计算.对于每一个do_test,用项目名称,输入,基于输入的期望结果来增加一个新的测试用例.
重新构建应用程序并切换到二进制目录,运行ctest可执行程序(ctest -N 或test -VV).对于多配置生成器(如Visual Studio),还必须指定配置类型.调试模式下,在构建目录下运行ctest -C Debug -VV.或者,在IDE中构建RUN_TESTS目标.

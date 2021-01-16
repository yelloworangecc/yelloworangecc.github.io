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

## 步骤5 增加系统自省 ##
考虑这种情况,我们向项目中添加一些代码,这些代码依赖的特性在目标平台不具备.例如,我们添加了一些依赖于log和exp函数的代码,目标平台是否支持这两个函数并不清楚.(当然,现实场景下几乎每一个平台都会提供这些函数.)
如果平台有log和exp函数,那么我们将在mysqrt函数中使用它们来计算平方根.首先我们在顶层CMakeLists.txt中使用CheckSymbolExists模块来测试函数是否可用.如果找不到函数,那么我们尝试在库中查找.
这种方式会在文件TutorialConfig.h.in中定义新的宏,所以确保以下语句位于设置配置文件语句前.
```
include(CheckSymbolExists)
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
if(NOT (HAVE_LOG AND HAVE_EXP))
  unset(HAVE_LOG CACHE)
  unset(HAVE_EXP CACHE)
  set(CMAKE_REQUIRED_LIBRARIES "m")
  check_symbol_exists(log "math.h" HAVE_LOG)
  check_symbol_exists(exp "math.h" HAVE_EXP)
  if(HAVE_LOG AND HAVE_EXP)
    target_link_libraries(MathFunctions PRIVATE m)
  endif()
endif()
```
接着,在文件TutorialConfig.h.in增加新的宏,这些宏会在源代码文件mysqrt.cxx中用到.
```
// 平台是否支持log和exp函数
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```
如果平台具备log和exp函数,我们将在mysqrt函数中使用它们来计算平方根.在MathFunctions/mysqrt.cxx的mysqrt函数中增加下面的代码.(不要忘记返回语句前的#endif)
```
#if defined(HAVE_LOG) && defined(HAVE_EXP)
  double result = exp(log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  double result = x;
```
我们还需要在源文件mysqrt.cxx中包含头文件cmath
```
#include <cmath>
```
运行cmake或cmake-gui配置项目,然后使用指定的构建工具执行构建动作并运行生成的可执行文件.
你将会注意到,我们并没有向期望的那样用到log和exp函数.原来我们在源文件mysqrt.cxx中忘记包含头文件TutorialConfig.h了.
此外,还需要更新MathFunctions/CMakeLists.txt文件,使源文件mysqrt.cxx能找到头文件TutorialConfig.h的位置.
```
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${CMAKE_BINARY_DIR}
          )
```
在更新完成后,重新构建并运行程序.看看哪个函数得到了更准确的结果?sqrt还是mysqrt?

### 指定编译宏 ###
有没有比头文件TutorialConfig.h更好的地方用来保存HAVE_LOG和HAVE_EXP宏?试试target_compile_definitions()函数吧.
首先从配置文件TutorialConfig.h.in中移除宏.源文件mysqrt.cxx也不再需要包含TutorialConfig.h头文件.
接着,将HAVE_LOG和HAVE_EXP宏移至MathFunctions/CMakeLists.txt并将它们指定为PRIVATE编译宏.
```
include(CheckSymbolExists)
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
if(NOT (HAVE_LOG AND HAVE_EXP))
  unset(HAVE_LOG CACHE)
  unset(HAVE_EXP CACHE)
  set(CMAKE_REQUIRED_LIBRARIES "m")
  check_symbol_exists(log "math.h" HAVE_LOG)
  check_symbol_exists(exp "math.h" HAVE_EXP)
  if(HAVE_LOG AND HAVE_EXP)
    target_link_libraries(MathFunctions PRIVATE m)
  endif()
endif()

# add compile definitions
if(HAVE_LOG AND HAVE_EXP)
  target_compile_definitions(MathFunctions
                             PRIVATE "HAVE_LOG" "HAVE_EXP")
endif()
```
更新完成后重新构建和运行可执行程序,检查运行结果.

## 步骤6 增加自定义命令和生成文件 ##
假设,为了实现这个教程的目标,我们决定不使用平台的log和exp函数,我们在mysqrt函数中使用一个预先计算好的数值表来代替.这一节,我们在构建过程中创建一个表,然后将表编译到应用程序中.
首先,移除MathFunctions/CMakeLists.txt中检查log和exp函数的内容.接着移除mysqrt.cxx中检查宏HAVE_LOG和HAVE_EXP的内容,同时移除cmath头文件包含语句.
在MathFunctions创建一个新的源文件用来生成数值表.
数值表是由合法的C++代码产生的,传入的参数是输出文件名.
下一步,在MathFunctions/CMakeLists.txt文件中添加一些用来构建可执行文件并在整个应用程序的构建过程中运行这个可执行程序的命令.
在MathFunctions/CMakeLists.txt文件的顶部,添加二进制MakeTable目标.
```
add_executable(MakeTable MakeTable.cxx)
```
继续添加自定义命令来控制Table.h的生成.
```
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
```
我们需要让CMake知道mysqrt.cxx依赖于生成的Table.将Table.h添加到MathFunctions库的源文件列表中.
```
add_library(MathFunctions
            mysqrt.cxx
            ${CMAKE_CURRENT_BINARY_DIR}/Table.h
            )
```
还需要将二进制生成目录添加到头文件目录列表,使Table.h能够被找到.
```
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${CMAKE_CURRENT_BINARY_DIR}
          )
```
现在我们来使用生成的数值表.首先,修改mysqrt.cxx包含头文件Table.h.接着,重写mysqrt函数来使用数值表.
```
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // 使用数值表来查找初始化值
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
  }

  // 10次迭代
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}
```
运行cmake或cmake-guil来配置工程并使用指定的构建工具执行构建动作.
在构建时,首先CMake首先构建MakeTable可执行文件,然后运行他来生成Table.h.最后,编译包含了Table.h头文件的mysqrt.cxx来生成MathFunctions库.
运行Tutorial可执行程序并验证它是否使用了数值表.

## 步骤7 构建一个安装器 ##
下一步,假设我们想要分发我们的项目给别人使用.我们想要在多种平台同时提供二进制和源代码.这与我们步骤4中描述的安装过程有一些区别.之前是先构建源代码然后安装得到的二进制文件.这个例子中,我们构建支持二进制安装和包管理的安装包.我们将会用到CPack来创建特定平台的安装器.特别地,我们需要向顶层CMakeLists.txt文件的底部添加下面内容.
```
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
```
这样就完成了!InstallRequiredSystemLibraries模块将会包含所有项目用到的当前平台的运行时库.接着,我们为工程设置了一些保存许可和版本信息的变量.版本信息在本教程的前面步骤已经设置过,这个步骤中我们在顶层源文件目录下添加了license.txt文件.
最后我们将CPack模块包含进来,CPack会用到上面的变量和当前系统的一些属性来设置一个安装器.
下一步,构建项目然后运行cpack可执行程序.构建二进制安装包的话,在二进制目录下运行`cpack`.
要指定生成器的话指定-G选项.对于多配置构建,使用-C选项来指定配置.下面是一个例子.
```
cpack -G ZIP -C Debug
```
构建源代码安装包的话运行下面的命令.
```
cpack --config CPackSourceConfig.cmake
```
也可以运行make package,或者在IDE中右击Package目标,点击Build Project.
在二进制目录下找到安装器,运行安装器可执行文件,检查它是否生效.

## 步骤8 增加对仪表盘的支持 ##
将我们的测试结果加入到仪表盘不是一个难事.在之前的步骤中我们已经定义了一些列的测试用例.现在我们只需要运行这些测试用例,并将它们提交到仪表盘.我们需要在顶层CMakeLists.txt中包含CTest模块来获得对仪表盘的支持.
用`include(CTest)`替换`enable_testing()`.CTest模块会自动调用enable_testing()函数.我们还需要在顶层目录下创建一个CTestConfig.cmake文件,这个文件用来设置工程的名称和仪表盘的位置.
```
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")

set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```
ctest可执行程序在运行时会读取这个文件.可以通过cmake或cmake-gui来为工程配置一个简单的仪表盘,先不要执行构建动作,进入二进制目录后运行`ctest [-VV] -D Experimental`.对于多配置生成器(比如Visual Studio),还必须指定配置类型.
```
ctest [-VV] -C Debug -D Experimental
```
如果是在IDE环境下,构建Experimental目标即可.
ctest可执行程序会构建并测试工程,然后将结果提交到[Kitware’s的公共仪表盘](https://my.cdash.org/index.php?project=CMakeTutorial).

## 步骤9 混合使用静态和动态库 ##
这一节我们学习如何使用BUILD_SHARED_LIBS变量改变add_library()函数的默认行为,以及无需显示指定(STATIC,SHARED,MODULE,OBJECT)的情况下控制库的构建类型.
我们需要在顶层CMakeLists.txt文件中增加BUILD_SHARED_LIBS变量.命令option()允许用户选择变量值为ON或OFF.
下一步我们重构MathFunctions,使其成为一个包装了mysqrt和sqrt的真正的库,不再需要调用代码中的控制逻辑.这也意味着USE_MYMATH将不再控制MathFunctions的构建过程,而是控制库的行为.
第一步,更新顶层CMakeLists.txt的开头部分.
```
cmake_minimum_required(VERSION 3.10)
# 设置工程名称和版本
project(Tutorial VERSION 1.0)
# 指定C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
# 设置静态和动态库的生成路径
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
# 配置一个头文件用来传入版本号
configure_file(TutorialConfig.h.in TutorialConfig.h)
# 添加MathFunctions库目标
add_subdirectory(MathFunctions)
# 添加一个可执行目标
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions)
```
既然现在我们总是使用MathFunctions库,那么我们需要相应地更新库的代码逻辑.在MathFunctions/CMakeLists.txt文件中,我们创建一个SqrtLibrary,它会在USE_MYMATH被使能的条件下被构建.在本教程中,我们显示地将SqrtLibrary构建为静态库.最终MathFunctions/CMakeLists.txt文件的内容如下.
```
# 添加一个库目标
add_library(MathFunctions MathFunctions.cxx)
# 使链接库的对象可以找到库的头文件
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )
# 是否使用自定义函数
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)
  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  # 添加用来生成数值表的可执行目标
  add_executable(MakeTable MakeTable.cxx)
  # 添加生成代码的命令
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    DEPENDS MakeTable
    )
  # 计算平方根的静态库
  add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )

  # Table.h的包含路径
  target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()

# 定义windows平台declspec(dllexport)相关的符号
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

# 安装规则
set(installable_libs MathFunctions)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```
下一步,向MathFunctions/mysqrt.cxx文件中添加mathfunctions和detail命名空间.
```
#include <iostream>

#include "MathFunctions.h"

// 包含生成的数值表
#include "Table.h"

namespace mathfunctions {
namespace detail {
// 一个简单的计算平方根的函数
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // 使用数值表来查找一个初始值
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
  }

  // 10次迭代
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}
}
}
```
我们还需要在tutorial.cxx文件中做一些修改:
1. 总是包含头文件MathFunctions.h
2. 总是使用mathfunctions::sqrt
3. 不在包含cmath
最后,更新MathFunctions/MathFunctions.h来使用dll导出定义.
```
#if defined(_WIN32)
#  if defined(EXPORTING_MYMATH)
#    define DECLSPEC __declspec(dllexport)
#  else
#    define DECLSPEC __declspec(dllimport)
#  endif
#else // non windows
#  define DECLSPEC
#endif

namespace mathfunctions {
double DECLSPEC sqrt(double x);
}
```
此时,如果我们开始编译的话,会出现链接错误,提示我们正在混合使用位置相关代码的静态库和位置无关代码的库.解决方法是不管构建类型为何,设置SqrtLibrary的属性POSITION_INDEPENDENT_CODE的值为真.
```
  # 当默认是共享库时,设置SqrtLibrary需要PIC
  set_target_properties(SqrtLibrary PROPERTIES
                        POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                        )

  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
```
练习:你能否通过查找CMake文档找到一个模块来简化我们在MathFunctions.h中添加的dll导出定义?

## 步骤10 生成表达式 ##
生成表达式用来在构建系统工作期间产生每一个构建配置对应的信息.
很多目标属性中可以指定生成表达式,比如LINK_LIBRARIES,INCLUDE_DIRECTORIES,COMPILE_DEFINITIONS等.还可以在命令中传递这些属性,比如target_link_libraries(),target_include_directories(),target_compile_definitions()等.
生成表达式也可以用来在编译过程中使能条件链接,条件宏定义,条件目录包含.这些控制条件可以基于构建配置,目标属性,平台信息,或者其他一些可查询到的信息.
生成表达是有几种不同类型,包括逻辑表达式,信息表达式和输出表达式.
逻辑表达式用来创建条件输出.最基本的表达式是0和1.<0:...>的结果是空字符串,<1:...>的结果是字符串的内容.它们是可以嵌套的.
生成表达式的一个常用用途是用来控制是否增加编译器选项,比如编译器的语言版本和告警层级.一个比较好的实践经验是将这项信息关联到INTERFACE目标,使这些信息可以继续传递.让我们构建一个INTERFACE目标,并制定C++版本为11,而不在使用CMAKE_CXX_STANDARD变量.将下面内容
```
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```
替换为
```
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
```
下一步,我们为工程添加一个想要的编译器告警选项.不同编译器的编译选项也不同,我们使用COMPILE_LANG_AND_ID生成表达式来控制在特定语言和编译器的情况下应该使用哪个选项.
```
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
```
我们可以看到告警选项包装在BUILD_INTERFACE条件中.这样安装我们工程的用户不会继承我们设置的告警选项.
练习:修改MathFunctions/CMakeLists.txt,是的所有目标都通过target_link_libraries()调用tutorial_compiler_flags.

## 步骤11 增加导出配置 ##
在步骤4安装和测试一节中,我们为CMake增加了安装库和头文件的能力.在步骤7构建安装器一节中,我们增加了打包能力,使得软件可以分发被其他人.
下一步,我们需要添加一些必要的信息,使其他CMake工程可以在构建目录,本地安装或安装包的情况下使用我们的工程.
首先,更新install(TARGETS)同时指定DESTINATION和EXPORT关键字.EXPORT关键字生成和安装一个cmake文件,这个文件包含了安装树上所有安装目标的导入代码.接着,更新MathFunctions/CMakeLists.txt文件,显示地导出MathFunctions库.
```
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs}
        DESTINATION lib
        EXPORT MathFunctionsTargets)
install(FILES MathFunctions.h DESTINATION include)
```
现在已经导出了MathFunctions,我们还需要显示的安装生成的MathFunctionsTargets.cmake文件.在顶层CMakeLists.txt的末尾添加下面的内容.
```
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
```
此时我们可以设置运行Cmake了.正常情况下你会看到CMake产生了下面的错误信息.
```
Target "MathFunctions" INTERFACE_INCLUDE_DIRECTORIES property contains
path:

  "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

which is prefixed in the source directory.
```
这个错误信息的含义是在在生成导出信息时,导出的路径是和当前机器固定关联的,在其他机器上是不合法的.解决方法是更新MathFunctions模板的target_include_directories()命令,使cmake能知道当在构建目录或安装包情况下被使用时需要一个不同的INTERFACE路径.这也意味着需要对target_include_directories()调用做如下修改.
```
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>
                           )
```
完成修改以后,重新运行CMake看是否还有告警.
此时,CMake将所需要的目标信息进行了打包,但是我们还是需要生成一个MathFunctionsConfig.cmake文件,这样CMake才能通过find_package()命令找到我们的工程.在工程的顶层目录下添加一个叫Config.cmake.in的新文件,文件内容如下.
```
@PACKAGE_INIT@

include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
```
下一步,为了正确地配置和安装这个文件,在顶层CMakeLists.txt的末尾添加如下内容.
```
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
include(CMakePackageConfigHelpers)
# 生成包含导出内容的配置文件
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
# 为配置文件生成版本文件
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)
# 安装配置文件
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  DESTINATION lib/cmake/MathFunctions
  )
```
此时,我们为工程生成了一个可以重定位的CMake配置,我们的工程因此可以在被安装或打包后被其他工程使用.如果想在构建目录下使用我们的工程,只需要在顶层CMakeLists.txt文件末尾添加下面内容.
```
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
```
这个导出调用语句可以生成一个Targets.cmake,这个文件允许其他工程在构建目录下使用配置的MathFunctionsConfig.cmake文件,这种情况下不需要安装工程.

## 步骤12 调试版本和发布版本的打包 ##
注意:这个例子只适用于单配置生成器,对多配置生成器无效(比如Visual Studio).
默认情况下,在CMake模型中,一个构建目录只包含一个配置,是Debug,Release,MinSizeRel或者RelWithDebInfo其中一个.然而CPack是可以将一个工程的多个构建目录(多个配置)捆绑在一起打成一个包的.
首先,我们需要确保调试版本和发布版本的可执行文件和库文件具有不同的名称.让我们为调试版本的可执行文件和库文件名称添加一个字母d后缀.
在顶层CMakeLists.txt的开始位置设置CMAKE_DEBUG_POSTFIX变量.
```
set(CMAKE_DEBUG_POSTFIX d)

add_library(tutorial_compiler_flags INTERFACE)
```
为可执行目标添加DEBUG_POSTFIX属性.
```
add_executable(Tutorial tutorial.cxx)
set_target_properties(Tutorial PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})

target_link_libraries(Tutorial PUBLIC MathFunctions)
```
为MatchFunctions库也添加一个版本号.在MathFunctions/CMakeLists.txt文件中设置VERSION和SOVERSION属性.
```
set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")
set_property(TARGET MathFunctions PROPERTY SOVERSION "1")
```
在顶层目录下创建debug和release两个子目录.
运行cmake命令,使用CMAKE_BUILD_TYPE选项指定配置类型.
```
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```
现在调试和发布版都构建完毕,我们使用一个自定义的配置文件将两个目录下的构建内容打包到一个发布版本包中.在工程顶层目录下创建一个MultiCPackConfig.cmake文件.在这个文件中,先包含cmake创建的默认配置文件.下一步,使用CPACK_INSTALL_CMAKE_PROJECTS变量指定哪一个工程会被安装.这个例子中,我们调试版本和发布版本都会被安装.
```
include("release/CPackConfig.cmake")

set(CPACK_INSTALL_CMAKE_PROJECTS
    "debug;Tutorial;ALL;/"
    "release;Tutorial;ALL;/"
    )
```
在工程顶层目录运行cpack命令并使用config选项指定我们自定义的配置文件.
```
cpack --config MultiCPackConfig.cmake
```
完成!

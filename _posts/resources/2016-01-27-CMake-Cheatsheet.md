---
layout: post
title:  "CMake CheatSheet"
date:   2016-01-29 11:30:00
tags: [cmake, cheatsheet]
categories: Resources
---
[Link1](http://name5566.com/1795.html) [Link2](http://www.hahack.com/codes/cmake/)

### 1. 常用:readlink获取工作目录全路径
{% highlight Bash shell scripts %}
WD=$(readlink -f "`dirname $0`")  # 取当前shell脚本的全路径，不包括最后的/
{% endhighlight %}
*注意：* 不要在cmake中用相对路径，用${CMAKE_CURRENT_SOURCE_DIR}等组合出全路径使用 

### 2. DEBUG or Release
Win和Linux系统，可以通过cmake -DCMAKE_BUILD_TYPE=Release/Debug设置编译目标

Xcode系统，上述变量可能没有效果，经测试可以使用CMAKE_CONFIGURATION_TYPES

### 3. 实例 
{% highlight Bash shell scripts %}
cmake_minimum_required (VERSION 3.1)  # 指定需要的 CMake 的最低版本
project (project)  # 指定项目的名称

# CMAKE_BINARY_DIR就是build目录，即这次build的顶层目录
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)  # 设置 ARCHIVE 目标的输出路径
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)  # 设置 LIBRARY 目标的输出路径
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)  # 设置 RUNTIME 目标的输出路径
set(CMAKE_CXX_STANDARD 11)  #  initialize the CXX_FLAGS on all targets, -std=c++11，一些C++高级特性

# [option](https://cmake.org/cmake/help/v3.4/command/option.html?highlight=option)
option(DEBUG "To log messges for debugging" 1)
if(DEBUG)
  add_definitions(-DDEBUG)
endif()
# Then, we can use "DEBUG=${DEBUG:-1}, cmake -DDEBUG=${DEBUG}" to turn on/off the flag

if(UNIX OR APPLE)  # UNIX-like 的系统，包括 Apple OS X 和 CygWin 或  Apple 系统
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -Wall -DUSE_OPENCV=1")
endif()
# 只将Release版本，设置为-Ofast
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Ofast")

# 输出信息：message([STATUS|WARNING|AUTHOR_WARNING|FATAL_ERROR|SEND_ERROR] “message to display” …)
message(STATUS "root path:${CMAKE_FIND_ROOT_PATH}")
# CMAKE_FIND_ROOT_PATH指定了一个或者多个优先于其他搜索路径的搜索路径。默认为空

include(cmake/other.cmake)  # Load and run CMake code from the file given

# CMake module path for custom module finding
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake)
# 定义自己的cmake子模块所在的路径，find_package来这里找，然后可以用INCLUDE命令来调用模块

if(DEFINED BUILD_ANDROID)  # 跨平台编译android程序，BUILD_ANDROID被android.toolchain.cmake置为true
  set(ENABLE_JNI 1)
  set(BUILD_SHARED_LIBS 1)  # 控制默认的库编译方式。若未设置,使用ADD_LIBRARY时又没指定库类型,默认生成静态库；设置1为动态库
  list(APPEND CMAKE_FIND_ROOT_PATH ${OWN_ANDROID_PATH}/)  # list命令，用来操作列表，CMAKE_FIND_ROOT_PATH查找路径
endif()

# find_package为外部工程加载设置
if(ENABLE_OPENCV)
   find_package(OpenCV REQUIRED)  # CMake搜索所有名为Find<package>.cmake的文件
# If no Find.cmake is found and the MODULE option is not given the command proceeds to Config mode:package-config.cmake类似文件
   include_directories(${OpenCV_INCLUDE_DIRS})  # 被编译器用来查找 include 文件
   message(STATUS "opencv:${OpenCV_INCLUDE_DIRS} ${OpenCV_LIBS} ")
   set(project_lib ${project_lib} ${OpenCV_LIBS})  # 存储lib到统一的变量里
endif()

# eigen
find_package(Eigen REQUIRED)
# 也可以使用target_include_directories，只为某一个target添加include目录
include_directories(SYSTEM ${EIGEN_INCLUDE_DIR})  # SYSTEM：to use system include directories on some platforms
add_definitions(-DUSE_EIGEN)  # 用于添加预编译，定义标志

# Qt5: [link](https://zhuanlan.zhihu.com/p/34667993)
find_package(Qt5 COMPONENTS Widgets REQUIRED)
# 如果提示找不到qt，请打印${CMAKE_PREFIX_PATH} ${Qt5_DIR}，分析下config file的路径中是否有Qt5WidgetsConfig.cmake
if (Qt5_FOUND)
  set(CMAKE_AUTOMOC ON)  # qt wrapper around c++, like QObject, CMAKE会自动生成moc文件
else()
  message(STATUS "QT not found, build without QT demo.")
endif()
# then we can use ${Qt5Widgets_INCLUDE_DIRS} and ${Qt5Widgets_LIBRARIES}

# caffe
# [link](https://cmake.org/cmake/help/v3.4/command/find_package.html?highlight=find_package)
find_package(Caffe REQUIRED)  # 若找到，则name_FOUND被自动置为1
find_library(CAFFE_LIB_PATH ${Caffe_LIBRARIES})  # 查找library的绝对路径，存入变量
message(STATUS "caffe:${Caffe_LIBRARIES} ${Caffe_FOUND} ${CAFFE_LIB_PATH}") 

# protobuf
find_package(Protobuf REQUIRED)
include_directories(${PROTOBUF_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR})
PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS myown.proto)
message(STATUS "pb:${PROTOBUF_LIBRARIES}")
# then we can add ${PROTO_SRCS} ${PROTO_HDRS} to add_library; and #include "myown.pb.h" in main.cpp

set(project_src a.cpp b.cpp c.cpp)

# 用于指定从一组源文件中编译出一个库文件 libproject.so
add_library(project ${OTHER_SRCS} ${project_src})
# 有STATIC/SHARED/OBJECT等方式(https://cmake.org/cmake/help/v3.4/command/add_library.html?highlight=add_library)
# OBJECT方式不创建实际的lib文件，跟直接src放进去相同，会初始化objs中的全局变量
# 用于指定project需要链接的库, 这里target必须已被创建，链接的item可以是已经存在的target（依赖关系会自动添加）
TARGET_LINK_LIBRARIES(project
        ${project_lib}
        ${CMAKE_OTHER_LIBS}
)

# 编译出可执行文件，就叫main
add_executable(main main.cpp )
set(main_lib project)
# main依赖于main_lib=project，就是libproject.so
TARGET_LINK_LIBRARIES(main ${main_lib})
# 如果运行时需要动态库，可以这么拷贝：copy the dlls to runtime dir
file(GLOB RUNTIME_DLLS "${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_ARCH}/*.dll")
foreach(file_i ${RUNTIME_DLLS})
  add_custom_command(TARGET main POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${file_i} ${CMAKE_BINARY_DIR}/)
endforeach()
# 如果copy目标文件夹不存在，则需要先新建
# COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/${dstdir}


if(ENABLE_JNI)
   add_subdirectory(src/jni)  # 添加一个需要进行构建的子目录，里边编辑子CMakeLists.txt
endif()

if(DEFINED BUILD_ANDROID)
   set(INSTALL_DIST_PATH ${PROJECT_SOURCE_DIR}/dist/android/${ANDROID_ABI})
else(NOT DEFINED BUILD_ANDROID)
   set(INSTALL_DIST_PATH ${PROJECT_SOURCE_DIR}/dist/${CMAKE_SYSTEM_NAME})  # CMAKE_SYSTEM_NAME=`uname -s`
endif(DEFINED BUILD_ANDROID)

install(TARGETS project DESTINATION ${INSTALL_DIST_PATH})  # 指定install的时候，执行的命令，跟make时没关系
# 将需要的lib也install到目标目录
# [link](https://cmake.org/cmake/help/v3.4/command/install.html#installing-files)
install(FILES ${CAFFE_LIB_PATH} DESTINATION ${INSTALL_DIST_PATH})

# install高级用法
# copy so/dlls to another project
if (UNIX AND NOT APPLE)
  # in linux docker: can change to other path to install
  if(NOT DEFINED Dst_Install_DIR)
    set(Dst_Install_DIR /home/a)
  endif()
  set(CMAKE_INSTALL_PREFIX ${Dst_Install_DIR})
  # 自动安装到lib/linux目录下
  install(TARGETS main_lib ARCHIVE DESTINATION lib/linux/
          LIBRARY DESTINATION lib/linux/)
  # Install headers
  Install(FILES "${project_DIR}/a.h" DESTINATION include/a)
elseif (MSVC)
  set(Dst_Install_DIR D:/Work/b)
  set(CMAKE_INSTALL_PREFIX ${Dst_Install_DIR})
  install(TARGETS main_lib ARCHIVE DESTINATION lib/${CMAKE_ARCH}/
          RUNTIME DESTINATION bin/${CMAKE_ARCH}/)
  # Install pdbs
  install(FILES $<TARGET_PDB_FILE:main_lib> DESTINATION lib/${CMAKE_ARCH}/ OPTIONAL)
  # Install headers
  Install(FILES "${project_DIR}/a.h" DESTINATION include)
else()  # mac
  set(Dst_Install_DIR /Users/b/b)
  set(CMAKE_INSTALL_PREFIX ${Dst_Install_DIR})
  install(TARGETS main_lib ARCHIVE DESTINATION lib/mac/
          LIBRARY DESTINATION lib/mac/)
  # Install headers
  Install(FILES "${project_DIR}/a.h" DESTINATION include/b)
endif()

# 设置一些Options. Turn on with 'cmake -Dmyvarname=ON'.
option(BUILD_TESTS "Build all tests." 0) # 可定义一些编译开关ON/OFF，最后给出默认值如0

########################
# 可以定义子函数
# Short command for adding unit-test
# Usage:
#     add_unit_test(test_case_name <src_file>)
########################
function(add_unit_test test_case_name)
  add_executable(${test_case_name} ${ARGN})  # ARGN隐式参数，如果是多个src_file参数

  # Standard linking to gtest stuff.
  target_link_libraries(${test_case_name} ${GTEST_BOTH_LIBRARIES})

  # Extra linking for the project.
  target_link_libraries(${test_case_name} ${main_lib})

  # This is so you can do 'make test' to see all your tests run, instead of
  # manually running the executable runUnitTests to see those specific tests.
  # [link](https://cmake.org/cmake/help/v3.4/command/add_test.html?highlight=add_test)
  add_test(NAME ${test_case_name} COMMAND ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${test_case_name})
endfunction()

if (BUILD_TESTS)
    enable_testing()
    # add_subdirectory(tests)  # 可在tests目录添加CMakeLists.txt, 并添加以下add_test等内容

    find_package(GTest REQUIRED)
    include_directories(${GTEST_INCLUDE_DIRS})

    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/test)  # 放到test目录

    # Add Unit Tests Here
    add_unit_test(TestName test/testMain.cpp src/a.cpp)
endif()
{% endhighlight %}

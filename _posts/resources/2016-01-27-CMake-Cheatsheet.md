---
layout: post
title:  "CMake CheatSheet"
date:   2016-01-29 11:30:00
tags: [cmake, cheatsheet]
categories: Resources
---
[Link](http://name5566.com/1795.html)

### 1. 常用:readlink获取工作目录全路径
{% highlight Bash shell scripts %}
WD=$(readlink -f "`dirname $0`")  # 取当前shell脚本的全路径，不包括最后的/
{% endhighlight %}

### 2. 实例 
{% highlight Bash shell scripts %}
cmake_minimum_required (VERSION 3.1)  # 指定需要的 CMake 的最低版本
project (project)  # 指定项目的名称

# CMAKE_BINARY_DIR就是build目录，即这次build的顶层目录
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)  # 设置 ARCHIVE 目标的输出路径
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)  # 设置 LIBRARY 目标的输出路径
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)  # 设置 RUNTIME 目标的输出路径
set(CMAKE_CXX_STANDARD 11)  #  initialize the CXX_FLAGS on all targets, -std=c++11，一些C++高级特性

# 设置一些Options. Turn on with 'cmake -Dmyvarname=ON'.
option(test "Build all tests." 0) # 可定义一些编译开关ON/OFF，最后给出默认值

if(UNIX OR APPLE)  # UNIX-like 的系统，包括 Apple OS X 和 CygWin 或  Apple 系统
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -Wall -DUSE_OPENCV=1")
endif()

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

#eigen
find_package(Eigen REQUIRED)
include_directories(SYSTEM ${EIGEN_INCLUDE_DIR})  # SYSTEM：to use system include directories on some platforms
add_definitions(-DUSE_EIGEN)  # 用于添加编译器命令行标志

#caffe
find_package(Caffe REQUIRED)  # [link](https://cmake.org/cmake/help/v3.4/command/find_package.html?highlight=find_package)若找到，则name_FOUND被自动置为1
find_library(CAFFE_LIB_PATH ${Caffe_LIBRARIES})  # 查找library的绝对路径，存入变量
message(STATUS "caffe:${Caffe_LIBRARIES} ${Caffe_FOUND} ${CAFFE_LIB_PATH}") 

set(project_src a.cpp b.cpp c.cpp)

# 用于指定从一组源文件中编译出一个库文件 libproject.so
add_library(project ${OTHER_SRCS} ${project_src})
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

if(ENABLE_JNI)
   add_subdirectory(src/jni)  # 添加一个需要进行构建的子目录
endif()

if(DEFINED BUILD_ANDROID)
   set(INSTALL_DIST_PATH ${PROJECT_SOURCE_DIR}/dist/android/${ANDROID_ABI})
else(NOT DEFINED BUILD_ANDROID)
   set(INSTALL_DIST_PATH ${PROJECT_SOURCE_DIR}/dist/${CMAKE_SYSTEM_NAME})  # CMAKE_SYSTEM_NAME=`uname -s`
endif(DEFINED BUILD_ANDROID)

install(TARGETS project DESTINATION ${INSTALL_DIST_PATH})  # 指定install的时候，执行的命令，跟make时没关系
# 将需要的lib也install到目标目录
install(FILES ${CAFFE_LIB_PATH} DESTINATION ${INSTALL_DIST_PATH}) #[link](https://cmake.org/cmake/help/v3.4/command/install.html#installing-files)
{% endhighlight %}
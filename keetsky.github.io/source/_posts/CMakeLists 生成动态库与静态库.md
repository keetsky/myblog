---
title: CMakeLists 生成动态库与静态库
date: 2018-02-10 20:02:19 
comments: true
english_title: CMakeLists   #对于中文标题需要个英文字体代替中文显示在URL上
categories:  #分类 
- ubuntu 
tags:	     #两个标签
- CMake

---
### 　CMakeLists 生成动态库与静态库

#### 文件结构

``` basic
├── bin
├── build
├── CMakeLists.txt
├── include
│   └── person.h
├── lib
└── src
    ├── CMakeLists.txt
    ├── main
    │   ├── CMakeLists.txt
    │   └── main.cpp
    └── person
        ├── CMakeLists.txt
        └── person.cpp
```
`最后生成的库文件将会放在lib目录下，并且生成的库文件带版本号`

-  **顶层CMakeLists.txt**
```
cmake_minimum_required(VERSION 3.3)
project(libraryTest CXX)
add_subdirectory(src)
```
-  **src/CMakeLists.txt**
```
# src CMakeLists.txt
add_subdirectory(main)
add_subdirectory(person)
```

-  **src/main/CMakeLists.txt**
```
# contacts CMakeLists.txt
aux_source_directory(. srcs)


include_directories(${PROJECT_SOURCE_DIR}/include  )

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

add_executable( librarytest ${srcs})

link_directories(${PROJECT_SOURCE_DIR}/lib)

target_link_libraries(librarytest
  person
  )
```
- **src/person/CMakeLists.txt**
```
aux_source_directory(. srcs)

include_directories(${PROJECT_SOURCE_DIR}/include )

set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# 之前的添加动态库/静态库的方法
# 缺点：动态库与静态库的名字不能重名
# add_library(person SHARED ${srcs})

# 生成动态库目标
add_library(person SHARED ${srcs})
# 生成静态库目标
add_library(person_static STATIC ${srcs})

# 指定静态库的输出名称
set_target_properties(person_static PROPERTIES OUTPUT_NAME "person")
# 使动态库和静态库同时存在
set_target_properties(person PROPERTIES CLEAN_DIRECT_OUTPUT 1)
set_target_properties(person_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

# 指定动态库版本
# VERSION 动态库版本
# SOVERSION API版本
set_target_properties(person PROPERTIES VERSION 1.0 SOVERSION 1)
```

- **运行**
```
$cd build
$cmake ..
$make
```







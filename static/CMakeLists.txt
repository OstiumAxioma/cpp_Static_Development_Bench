cmake_minimum_required(VERSION 3.20)
project(TemplateLib CXX)

# 设置C++标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 解决MSVC编译器字符编码警告
if(MSVC)
    add_compile_options("/source-charset:utf-8")
endif()

# VTK配置 - 版本8.2.0
set(VTK_DIR "D:/code/vtk8.2.0/VTK-8.2.0/lib/cmake/vtk-8.2")
find_package(VTK 8.2 REQUIRED)

# 包含VTK使用文件（VTK 8.2必需）
include(${VTK_USE_FILE})

# 设置输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# 收集源文件
set(SOURCES
    src/template.cpp
)

set(HEADERS
    header/template.h
)

# 创建静态库
add_library(TemplateLib STATIC ${SOURCES} ${HEADERS})

# 设置包含目录
target_include_directories(TemplateLib PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/header
    ${VTK_INCLUDE_DIRS}
)

# 链接VTK库
target_link_libraries(TemplateLib PUBLIC ${VTK_LIBRARIES})

# Windows特定设置
if(WIN32)
    # 定义预处理器宏
    target_compile_definitions(TemplateLib PRIVATE
        _WINDOWS
        _UNICODE
        UNICODE
    )
    
    # 设置编译选项
    target_compile_options(TemplateLib PRIVATE
        /W3     # 警告级别3
        /EHsc   # 启用C++异常处理
        /MP     # 多处理器编译
    )
    
    # Release配置优化
    target_compile_options(TemplateLib PRIVATE
        $<$<CONFIG:Release>:/O2>    # 最大优化
        $<$<CONFIG:Release>:/GL>    # 全程序优化
    )
    
    # Debug配置
    target_compile_options(TemplateLib PRIVATE
        $<$<CONFIG:Debug>:/Od>      # 禁用优化
        $<$<CONFIG:Debug>:/Zi>      # 生成调试信息
    )
endif()

# 显示配置信息
message(STATUS "Building TemplateLib Static Library with VTK support")
message(STATUS "VTK_VERSION: ${VTK_VERSION}")
message(STATUS "VTK_USE_FILE: ${VTK_USE_FILE}")
message(STATUS "Output directory: ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}")
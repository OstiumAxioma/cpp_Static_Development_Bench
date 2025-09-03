cmake_minimum_required(VERSION 3.14)
project(VTK_Qt_Project LANGUAGES CXX)

# 生成编译数据库供IDE使用
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# 设置C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Exe)

# Qt设置
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOUIC_SEARCH_PATHS ${CMAKE_CURRENT_SOURCE_DIR})

# 设置Qt路径
set(Qt5_DIR "C:/Qt/Qt5.12.9/5.12.9/msvc2017_64/lib/cmake/Qt5")
set(CMAKE_PREFIX_PATH "C:/Qt/Qt5.12.9/5.12.9/msvc2017_64")

# 查找Qt包
find_package(Qt5 COMPONENTS Core Gui Widgets REQUIRED)

# VTK配置 - 版本8.2.0 (新路径)
set(VTK_DIR "D:/code/vtk8.2.0/VTK-8.2.0/lib/cmake/vtk-8.2")
find_package(VTK 8.2 REQUIRED)

# VTK Qt支持检测和配置
if(VTK_QT_FOUND)
    message(STATUS "VTK Qt支持已找到")
else()
    set(VTK_QT_FOUND TRUE)
    message(STATUS "VTK Qt支持已启用")
endif()

# UI文件设置
set(UI_DIR ${CMAKE_CURRENT_SOURCE_DIR}/header)
set(CMAKE_AUTOUIC_SEARCH_PATHS ${CMAKE_CURRENT_SOURCE_DIR})

# 源文件
set(SOURCES
    src/main.cpp
    src/mainwindow.cpp
)

# 头文件
set(HEADERS
    header/mainwindow.h
)

# UI文件
set(FORMS
    src/mainwindow.ui
)

# 添加可执行文件
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS} ${FORMS})

# 添加包含目录
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/header
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/static/header  # 添加静态库头文件目录
)

# 设置编译定义
target_compile_definitions(${PROJECT_NAME} PRIVATE
    QT_DEPRECATED_WARNINGS
)

# 设置MSVC编译器UTF-8支持
if(MSVC)
    # 设置源文件字符集为UTF-8，解决中文字符编译警告
    add_compile_options("/source-charset:utf-8")
    # 同时设置执行字符集为UTF-8
    target_compile_options(${PROJECT_NAME} PRIVATE /utf-8)
endif()

# 查找静态库文件
find_library(TEMPLATE_LIB_RELEASE 
    NAMES TemplateLib
    PATHS ${CMAKE_CURRENT_SOURCE_DIR}/lib
    NO_DEFAULT_PATH
)

find_library(TEMPLATE_LIB_DEBUG
    NAMES TemplateLib_d
    PATHS ${CMAKE_CURRENT_SOURCE_DIR}/lib
    NO_DEFAULT_PATH
)

# 检查是否找到静态库
if(NOT TEMPLATE_LIB_RELEASE)
    message(WARNING "TemplateLib.lib not found in ${CMAKE_CURRENT_SOURCE_DIR}/lib")
endif()

if(NOT TEMPLATE_LIB_DEBUG)
    message(WARNING "TemplateLib_d.lib not found in ${CMAKE_CURRENT_SOURCE_DIR}/lib")
endif()

# 链接库
target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    ${VTK_LIBRARIES}
)

# 根据配置链接对应的静态库
if(TEMPLATE_LIB_RELEASE OR TEMPLATE_LIB_DEBUG)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:${TEMPLATE_LIB_DEBUG}>
        $<$<CONFIG:Release>:${TEMPLATE_LIB_RELEASE}>
    )
else()
    message(FATAL_ERROR "Static library TemplateLib not found!")
endif()

# 部署Qt DLL
if(WIN32)
    # 找到windeployqt.exe
    get_target_property(_qmake_executable Qt5::qmake IMPORTED_LOCATION)
    get_filename_component(_qt_bin_dir "${_qmake_executable}" DIRECTORY)
    find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS "${_qt_bin_dir}")
    
    # 添加命令在构建后运行windeployqt
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND "${WINDEPLOYQT_EXECUTABLE}"
            --verbose 0
            --no-translations
            --no-compiler-runtime
            --no-opengl-sw
            "$<TARGET_FILE:${PROJECT_NAME}>"
        COMMENT "正在部署Qt..."
    )
endif()
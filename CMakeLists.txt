cmake_minimum_required(VERSION 3.5)
project(ji)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_INSTALL_PREFIX "/usr/local/ev_sdk")
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
SET(CMAKE_BUILD_TYPE "Release")

#先编译三方库中的源码，并包含头文件路径
include_directories(${CMAKE_SOURCE_DIR}/3rd/license/include)
link_directories(${CMAKE_SOURCE_DIR}/3rd/license/lib/)

#jsoncpp相关头文件路径和源文件,除授权库以外的其他地方使用json是都是用jsoncpp来实现
include_directories(${CMAKE_SOURCE_DIR}/3rd/cJSON/include)
add_subdirectory(${CMAKE_SOURCE_DIR}/3rd/cJSON)

#cJson相关头文件路径和源文件,由于授权库内部使用到,所以需要编译链接cJson
include_directories(${CMAKE_SOURCE_DIR}/3rd/jsoncpp_simple/include)
add_subdirectory(${CMAKE_SOURCE_DIR}/3rd/jsoncpp_simple)

#配置点线框解析相关头文件路径和源文件
include_directories(${CMAKE_SOURCE_DIR}/3rd/wkt_parser/include)
add_subdirectory(${CMAKE_SOURCE_DIR}/3rd/wkt_parser)


# 使用pkg-config查找必要的依赖包
find_package(OpenCV REQUIRED)
find_library(glog REQUIRED)
set(ENV{PKG_CONFIG_PATH} "ENV{PKG_CONFIG_PATH}")
find_package(PkgConfig REQUIRED)
set(onnxruntime_INCLUDE_DIRS "/usr/local/ev_sdk/3rd/onnxruntime-linux-x64-1.6.0/include") #本demo所需的onnxruntime
include_directories(${onnxruntime_INCLUDE_DIRS})

set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_RPATH}:/usr/local/ev_sdk/lib")

# 编译libji.so
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRCS)
list(APPEND libji_SOURCES  ${SRCS})

add_library(${PROJECT_NAME} SHARED ${libji_SOURCES})
target_include_directories(${PROJECT_NAME}
        PUBLIC ${OpenCV_INCLUDE_DIR}
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )

target_link_libraries(${PROJECT_NAME}
        ${OpenCV_LIBS}
        glog
        WKTParser
        jsoncpp   
        ji_license
        ji_license_impl
        cjson
        "/usr/local/ev_sdk/3rd/onnxruntime-linux-x64-1.6.0/lib/libonnxruntime.so"        
        )
# ---  END EV_SDK DEPENDENCIES  ---

#安装算法运行所需要的库到对应的目录
# 安装SampleDetector所依赖的onnxruntime库, 请在此处将自己依赖的库安装到/usr/local/ev_sdk/lib下。此处需注意，务必将相关依赖库都安装在该目录下，
# 避免运行时无法找到链接库（算法sdk被上层应用调用时依赖该/usr/local/ev_sdk/lib路径）
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/3rd/onnxruntime-linux-x64-1.6.0/lib/libonnxruntime.so DESTINATION lib)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/3rd/onnxruntime-linux-x64-1.6.0/lib/libonnxruntime.so.1.6.0 DESTINATION lib)
#安装授权库相关的文件
install(DIRECTORY ${CMAKE_SOURCE_DIR}/3rd/license/lib/ DESTINATION lib)
# Install libji.so
install(TARGETS ${PROJECT_NAME} DESTINATION lib)
install(DIRECTORY ${CMAKE_SOURCE_DIR}/3rd/fonts DESTINATION lib)


# 编译 JI API 测试程序
add_executable(test-ji-api ${CMAKE_CURRENT_SOURCE_DIR}/test/test.cpp)
target_link_libraries(test-ji-api
        ${OpenCV_LIBS}
        cjson
        glog
        ${PROJECT_NAME}
        )
install(TARGETS test-ji-api DESTINATION bin)        
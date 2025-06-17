cmake_minimum_required(VERSION 3.9)
project(concurrentqueue VERSION 1.0.0)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# 这是生成了一个纯接口库，所以不包含任何源码，只是提供了头文件的搜索路径，保存在三个生成的 .cmake  文件中
add_library(${PROJECT_NAME} INTERFACE)

# 为 ${PROJECT_NAME} 动态设置头文件搜索目录。
target_include_directories(${PROJECT_NAME} 
        INTERFACE 
                # 开发/构建时候使用,由于开发的时候源文件保存在当前目录，所以使用当前目录作为头文件搜索路径
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>   
                # 安装后被其它文件应用的时候，由于已经进入到了linux系统的系统级目录，所以使用标准安装路径
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/> # 安装后被其他项目引用
)
# 这里是声明
install(TARGETS ${PROJECT_NAME} 
    EXPORT ${PROJECT_NAME}Targets
)


# 这个 CMake 命令 write_basic_package_version_file 用于生成一个 包版本配置文件（<PackageName>ConfigVersion.cmake），是 CMake 包管理系统的核心组件之一。
# 它的核心作用是实现版本兼容性检查，确保其他项目通过 find_package 查找该包时，能够正确匹配版本要求。
# 最后生成了一个版本配置文件 ${PROJECT_NAME}ConfigVersion.cmake，在该文件中，其实是一个if 语句进行，版本检查。当find_library的时候，会自动的导入这个脚本文件，进行检查。

write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    VERSION 
        ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
    ARCH_INDEPENDENT
)

# <PackageName>Config.cmake 作为主入口文件
configure_package_config_file(${PROJECT_NAME}Config.cmake.in
                ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        INSTALL_DESTINATION 
                ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}/
)

install(EXPORT 
                ${PROJECT_NAME}Targets
        FILE
                ${PROJECT_NAME}Targets.cmake
        NAMESPACE 
                "${PROJECT_NAME}::"
        DESTINATION
                ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
        COMPONENT
                Devel
)

install(
        FILES
                ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
                ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION
                ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
        COMPONENT
                Devel
)

install(
        FILES 
                blockingconcurrentqueue.h 
                concurrentqueue.h 
                lightweightsemaphore.h 
                LICENSE.md
        DESTINATION 
                ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/moodycamel
)

set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "Cameron Desrochers <cameron@moodycamel.com>")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "An industrial-strength lock-free queue for C++.")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER ${CPACK_PACKAGE_VENDOR})
set(CPACK_GENERATOR "RPM;DEB")

# 在这里会根据之前设置的变量，通过 include(CPack) 命令激活。
include(CPack)

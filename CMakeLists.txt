
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(ZeekPluginDPDK)

include(ZeekPlugin)

zeek_plugin_begin(ESnet DPDK)
zeek_plugin_cc(src/Plugin.cc src/DPDK.cc)
zeek_plugin_dist_files(README CHANGES COPYING VERSION)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-march=native" COMPILER_SUPPORTS_MARCH_NATIVE)
if(COMPILER_SUPPORTS_MARCH_NATIVE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
endif()

find_package(PkgConfig REQUIRED)
if(NOT PKG_CONFIG_FOUND)
    message(FATAL_ERROR "pkg-config not found!" )
endif()

pkg_check_modules(libdpdk REQUIRED IMPORTED_TARGET libdpdk)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${libdpdk_LIBRARIES_STATIC}")

set(CMAKE_CXX_LINK_EXECUTABLE "${CMAKE_CXX_LINK_EXECUTABLE} ${libdpdk_LIBRARIES_STATIC}")

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${libdpdk_LIBRARIES_STATIC}")

add_link_options("${libdpdk_LINK_LIBRARIES}")
#message(INFO "${libdpdk_LINK_LIBRARIES}")
zeek_plugin_link_library("${libdpdk_LINK_LIBRARIES}")

zeek_plugin_end()
include_directories(${libdpdk_INCLUDE_DIRS})

file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/VERSION" VERSION LIMIT_COUNT 1)

if ("${PROJECT_SOURCE_DIR}" STREQUAL "${CMAKE_SOURCE_DIR}")
    # Allows building rpm/deb packages via "make package" in build dir.
    include(ConfigurePackaging)
    ConfigurePackaging(${VERSION})
endif ()

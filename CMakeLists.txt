cmake_minimum_required(VERSION 3.24)
project(skins_generator)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 3)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_RELEASE_TYPE "release")
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}-${PROJECT_RELEASE_TYPE}")

# 生成 config.h
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/config.h"
)

# 收集头文件和源码文件
file(GLOB HEADER_FILES include/*.h)
file(GLOB SOURCE_FILES src/*.cpp)
message(STATUS "HEADER_FILES: ${HEADER_FILES}")
message(STATUS "SOURCE_FILES: ${SOURCE_FILES}")

include_directories(${CMAKE_SOURCE_DIR}/include ${CMAKE_CURRENT_BINARY_DIR})

# 拷贝资源目录
file(REMOVE_RECURSE "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/resource")
file(COPY
    "${CMAKE_CURRENT_SOURCE_DIR}/resource"
    DESTINATION "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}"
)

# 查找 Boost
add_definitions(-DBOOST_USE_WINAPI_VERSION=0x0600)
find_package(Boost 1.83 REQUIRED COMPONENTS log log_setup)
if (Boost_FOUND)
    message(STATUS "Boost library status:")
    message(STATUS "    Boost_VERSION: ${Boost_VERSION}")
    message(STATUS "    Boost_INCLUDE_DIRS: ${Boost_INCLUDE_DIRS}")
    message(STATUS "    Boost_LIBRARIES: ${Boost_LIBRARIES}")
else ()
    message(FATAL_ERROR "Boost not found.")
endif ()

# 查找 ZLIB
find_package(ZLIB REQUIRED)
if (ZLIB_FOUND)
    message(STATUS "ZLIB version: ${ZLIB_VERSION_STRING}")
else ()
    message(FATAL_ERROR "ZLIB not found.")
endif ()

# 查找 libzip
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBZIP REQUIRED libzip)
if (LIBZIP_FOUND)
    message(STATUS "LibZip version: ${LIBZIP_VERSION}")
else ()
    message(FATAL_ERROR "LibZip not found.")
endif ()

include_directories(${LIBZIP_INCLUDE_DIRS})
link_directories(${LIBZIP_LIBRARY_DIRS})

# 可执行文件
add_executable(skins_generator ${SOURCE_FILES} ${HEADER_FILES})
target_compile_options(skins_generator PUBLIC -O2)

# 链接库
target_link_libraries(skins_generator
    ${Boost_LIBRARIES}
    ${LIBZIP_LIBRARIES}
    ZLIB::ZLIB
)

cmake_minimum_required(VERSION 3.24)
project(skins_generator)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 3)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_RELEASE_TYPE "release")
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}-${PROJECT_RELEASE_TYPE}")

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/config.h"
)

# Collect source and header files
file(GLOB HEADER_FILES include/*.h)
file(GLOB SOURCE_FILES src/*.cpp)
message(STATUS "HEADER_FILES: ${HEADER_FILES}")
message(STATUS "SOURCE_FILES: ${SOURCE_FILES}")

include_directories(${CMAKE_SOURCE_DIR}/include ${CMAKE_CURRENT_BINARY_DIR})

message(STATUS "CMAKE_RUNTIME_OUTPUT_DIRECTORY: ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
file(REMOVE_RECURSE "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/resource")
file(COPY
    "${CMAKE_CURRENT_SOURCE_DIR}/resource"
    DESTINATION "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}"
)

# Dependencies (CMake >= 3.31, via vcpkg CONFIG mode)
find_package(Boost.Log CONFIG REQUIRED)
find_package(Boost.LogSetup CONFIG REQUIRED)
find_package(ZLIB REQUIRED)
find_package(libzip CONFIG REQUIRED)

add_executable(skins_generator src/Generator.cpp ${HEADER_FILES} ${SOURCE_FILES})
target_compile_options(skins_generator PRIVATE -O2)

target_link_libraries(skins_generator
    PRIVATE
        Boost::log
        Boost::log_setup
        libzip::zip
        ZLIB::ZLIB
)

# Optionally copy DLL dependencies to output (for local dev only; not needed in CI)
# add_custom_command(
#     TARGET skins_generator
#     POST_BUILD
#     COMMAND ${CMAKE_COMMAND} -E copy_if_different
#         "$<TARGET_FILE:ZLIB::ZLIB>"
#         "$<TARGET_FILE_DIR:skins_generator>"
#     COMMAND ${CMAKE_COMMAND} -E copy_if_different
#         "$<TARGET_FILE:libzip::zip>"
#         "$<TARGET_FILE_DIR:skins_generator>"
# )

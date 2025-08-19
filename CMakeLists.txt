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

# Find dependencies using vcpkg CONFIG mode (CMake >= 3.31 compatible)
find_package(boost_log CONFIG REQUIRED)
find_package(boost_log_setup CONFIG REQUIRED)
find_package(ZLIB REQUIRED)
find_package(libzip CONFIG REQUIRED)

add_executable(skins_generator src/Generator.cpp ${HEADER_FILES} ${SOURCE_FILES})
target_compile_options(skins_generator PRIVATE -O2)

target_link_libraries(skins_generator
    PRIVATE
        boost_log::boost_log
        boost_log_setup::boost_log_setup
        libzip::zip
        ZLIB::ZLIB
)

# Optionally copy DLL dependencies to output (if needed, for local dev)
# Uncomment if you want post-build DLL copy, but not needed for CI usually
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

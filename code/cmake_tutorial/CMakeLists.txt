cmake_minimum_required(VERSION 3.16)
project(cmake_tutorial LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(SRC "${PROJECT_SOURCE_DIR}/src/")

file(GLOB_RECURSE PRJ_SRC "${SRC}*.cpp")

add_executable(${PROJECT_NAME} 
    ${PRJ_SRC}
)

include_directories(${PROJECT_NAME} PRIVATE 
    ${SRC}
)
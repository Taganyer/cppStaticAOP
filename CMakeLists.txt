cmake_minimum_required(VERSION 3.29)

project(cppStaticAOP CXX)

set(CMAKE_CXX_STANDARD 17)

add_subdirectory(AOP_src)

# you can delete it.
add_subdirectory(AOP_test)

add_executable(${PROJECT_NAME} main.cpp)

target_link_libraries(${PROJECT_NAME} AOP_src)

# you can delete it.
target_link_libraries(${PROJECT_NAME}  AOP_test)

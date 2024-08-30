#项目名
project(AOP_test CXX)

set(src_list AOP_test.hpp AOP_test.cpp)

# 创建 library
add_library(${PROJECT_NAME} ${src_list})

# 获取当前目录的绝对路径
get_filename_component(current_dir ${CMAKE_CURRENT_SOURCE_DIR} ABSOLUTE)

# 指定当前模块的头文件搜索路径
target_include_directories(${PROJECT_NAME} PUBLIC "${current_dir}/..")

set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE CXX)

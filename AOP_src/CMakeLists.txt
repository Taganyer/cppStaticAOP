#项目名
project(AOP_src CXX)

# 项目源文件和头文件列表（考虑到 IDE 的分析功能，故加入头文件）
set(src_list AOP.hpp SourceLocation.hpp)

# 创建 library
add_library(${PROJECT_NAME} ${src_list})

# 获取当前目录的绝对路径
get_filename_component(current_dir ${CMAKE_CURRENT_SOURCE_DIR} ABSOLUTE)

# 指定当前模块的头文件搜索路径
target_include_directories(${PROJECT_NAME} PUBLIC "${current_dir}/..")

set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE CXX)

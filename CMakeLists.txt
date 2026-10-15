cmake_minimum_required(VERSION 3.28.3)
project(CAPD-Py LANGUAGES CXX)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_BUILD_TYPE Release)

# add_compile_options(-fsanitize=address)
# add_link_options(-fsanitize=address)

add_subdirectory(lib/CAPD)

add_subdirectory(lib/pybind11)
pybind11_add_module(capdpy src/capdpy.cpp)

target_link_libraries(capdpy PRIVATE capd filib)


install(TARGETS capdpy
        LIBRARY DESTINATION .)
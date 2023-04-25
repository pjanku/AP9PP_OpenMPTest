cmake_minimum_required(VERSION 3.25)
project(OpenMPTest)

find_package(OpenMP)

set(CMAKE_CXX_STANDARD 17)

add_executable(OpenMPTest main.cpp)
if(OpenMP_CXX_FOUND)
    target_link_libraries(OpenMPTest PUBLIC OpenMP::OpenMP_CXX)
endif()
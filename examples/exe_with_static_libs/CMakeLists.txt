cmake_minimum_required(VERSION 3.0)

#project(exe_with_static_libs)
SET ( TARGET_NAME exe_with_static_libs )
MESSAGE( STATUS "configuring ${TARGET_NAME}" )

include(../../scripts/cmake/common.cmake)

include_directories(../../doctest/)

#== create object libraries instead of static libraries
doctest_add_library(lib_1 OBJECT lib_1_src1.cpp lib_1_src2.cpp)
doctest_add_library(lib_2 OBJECT lib_2_src.cpp)
doctest_add_executable(${TARGET_NAME} main.cpp $<TARGET_OBJECTS:lib_1> $<TARGET_OBJECTS:lib_2>)

#== alternatively we could create static libraries and use "doctest_force_link_static_lib_in_target"
#doctest_add_library(lib_1 STATIC lib_1_src1.cpp lib_1_src2.cpp)
#doctest_add_library(lib_2 STATIC lib_2_src.cpp)
#doctest_add_executable(${TARGET_NAME} main.cpp)
#target_link_libraries(${TARGET_NAME} lib_1)
#target_link_libraries(${TARGET_NAME} lib_2)
#include(doctest_force_link_static_lib_in_target.cmake)
#doctest_force_link_static_lib_in_target(${TARGET_NAME} lib_1)
#doctest_force_link_static_lib_in_target(${TARGET_NAME} lib_2)

#== group them together in a single folder inside IDEs
set_target_properties(lib_1 PROPERTIES FOLDER "3rdParty/doctest/Examples/${TARGET_NAME}")
set_target_properties(lib_2 PROPERTIES FOLDER "3rdParty/doctest/Examples/${TARGET_NAME}")
set_target_properties(${TARGET_NAME} PROPERTIES FOLDER "3rdParty/doctest/Examples/${TARGET_NAME}")

doctest_add_test(NAME ${TARGET_NAME} COMMAND $<TARGET_FILE:${TARGET_NAME}> --no-version)

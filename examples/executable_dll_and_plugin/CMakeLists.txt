cmake_minimum_required(VERSION 3.0)

#project(executable_dll_and_plugin)
SET ( TARGET_NAME executable_dll_and_plugin )
MESSAGE( STATUS "configuring ${TARGET_NAME}" )

include(../../scripts/cmake/common.cmake)

include_directories(../../doctest/)

if(APPLE)
    set(CMAKE_MACOSX_RPATH ON)
    if(DEFINED ENV{TRAVIS})
        # force 64 bit for OSX because -m32 fails >>> SOMETIMES <<< (x86_64 vs 386 - like the dll is not linked with -m32...)
        add_compiler_flags(-m64)
    endif()
endif()

doctest_add_library(implementation SHARED implementation.cpp implementation_2.cpp)

doctest_add_library(dll SHARED dll.cpp)
target_link_libraries(dll implementation)

doctest_add_library(plugin SHARED plugin.cpp)
target_link_libraries(plugin implementation)

doctest_add_executable(${TARGET_NAME} main.cpp)
target_link_libraries(${TARGET_NAME} dll)
target_link_libraries(${TARGET_NAME} implementation)

if(NOT WIN32)
    target_link_libraries(${TARGET_NAME} dl)
endif()

# have the executable depend on the plugin so it gets built as well when building/starting only the executable
add_dependencies(${TARGET_NAME} plugin)

#== group them together in a single folder inside IDEs
set_target_properties(implementation PROPERTIES FOLDER "3rdParty/doctest/Examples/${TARGET_NAME}")
set_target_properties(dll PROPERTIES FOLDER "3rdParty/doctest/Examples/${TARGET_NAME}")
set_target_properties(plugin PROPERTIES FOLDER "3rdParty/doctest/Examples/${TARGET_NAME}")
set_target_properties(${TARGET_NAME} PROPERTIES FOLDER "3rdParty/doctest/Examples/${TARGET_NAME}")

doctest_add_test(NAME ${TARGET_NAME} COMMAND $<TARGET_FILE:${TARGET_NAME}> --no-version)

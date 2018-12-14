cmake_minimum_required(VERSION 3.0)
if(NOT DEFINED TARGET_NAME)
  set(NOT_SUBPROJECT ON)
endif()
SET ( TARGET_NAME doctest )
MESSAGE( STATUS "configuring ${TARGET_NAME}" )
################################################################################
## DOCTEST
################################################################################

file(READ ${CMAKE_CURRENT_SOURCE_DIR}/scripts/version.txt ver)
if (NOT NOT_SUBPROJECT)
	project(doctest VERSION ${ver})
endif ()

option(DOCTEST_WITH_TESTS               "Build tests/examples" ${NOT_SUBPROJECT})
option(DOCTEST_WITH_MAIN_IN_STATIC_LIB  "Build a static lib (cmake target) with a main entry point" OFF)
option(DOCTEST_NO_INSTALL  "Skip the installation process" ON)

add_library(${TARGET_NAME} INTERFACE)
target_include_directories(${TARGET_NAME} INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/doctest/>)

################################################################################
## TESTS/EXAMPLES/HELPERS
################################################################################

if(${DOCTEST_WITH_MAIN_IN_STATIC_LIB})
    include(scripts/cmake/common.cmake)
    doctest_add_library(${TARGET_NAME}_with_main STATIC EXCLUDE_FROM_ALL doctest/parts/doctest_main.cpp)
    target_link_libraries(${TARGET_NAME}_with_main PUBLIC ${TARGET_NAME})
endif()

if(${DOCTEST_WITH_TESTS})
    include(scripts/cmake/common.cmake)

    add_subdirectory(examples/all_features)
    
    if(NOT DEFINED ENV{CODE_COVERAGE})
        add_subdirectory(examples/exe_with_static_libs)
        add_subdirectory(examples/executable_dll_and_plugin)
        add_subdirectory(scripts/playground)
    endif()
endif()

################################################################################
## PACKAGE SUPPORT
################################################################################

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(include_install_dir "include/doctest/")
set(config_install_dir "lib/cmake/${TARGET_NAME}")

set(version_config "${generated_dir}/${TARGET_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${TARGET_NAME}Config.cmake")
set(targets_export_name "${TARGET_NAME}Targets")
set(namespace "${TARGET_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" VERSION ${ver} COMPATIBILITY SameMajorVersion
)

configure_file("scripts/cmake/Config.cmake.in" "${project_config}" @ONLY)

if (NOT ${DOCTEST_NO_INSTALL})
    install(
        TARGETS ${TARGET_NAME}
        EXPORT "${targets_export_name}"
        INCLUDES DESTINATION "${include_install_dir}"
    )

    install(
        FILES "doctest/doctest.h"
        DESTINATION "${include_install_dir}"
    )

    install(
        FILES "${project_config}" "${version_config}"
        DESTINATION "${config_install_dir}"
    )

    install(
        EXPORT "${targets_export_name}"
        NAMESPACE "${namespace}"
        DESTINATION "${config_install_dir}"
    )
endif()
MESSAGE( STATUS "${TARGET_NAME} configured" )
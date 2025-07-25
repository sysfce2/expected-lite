# Copyright 2016-2018 by Martin Moene
#
# https://github.com/martinmoene/expected-lite
#
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

cmake_minimum_required( VERSION 3.15 FATAL_ERROR )

# expected-lite project and version, updated by script/update-version.py:

project(
    expected_lite
    VERSION 0.9.0
    LANGUAGES CXX )

set(PROJECT_DESCRIPTION "Expected objects in C++11 and later in a single-file header-only library")
set(PROJECT_HOMEPAGE_URL "https://github.com/martinmoene/expected-lite")

# Package information:

set( unit_name       "expected" )
set( package_nspace  "nonstd" )
set( package_name    "${unit_name}-lite" )
set( package_version "${${PROJECT_NAME}_VERSION}" )

set( CPACK_PACKAGE_NAME "expected-lite" )
set( CPACK_PACKAGECPACK_PACKAGE_DESCRIPTION "${PROJECT_DESCRIPTION}" )
set( CPACK_DEBIAN_PACKAGE_NAME "lib${CPACK_PACKAGE_NAME}-dev" )
set( CPACK_RPM_PACKAGE_NAME "lib${CPACK_PACKAGE_NAME}-devel" )
set( CPACK_PACKAGE_HOMEPAGE_URL "${PROJECT_HOMEPAGE_URL}" )
set( CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all" )
set( CPACK_PACKAGE_MAINTAINER "Martin Moene" )
set( CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_MAINTAINER}" )
set( CPACK_DEBIAN_COMPRESSION_TYPE "xz" )
set( CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt" )
set( CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md" )
set( CPACK_PACKAGE_DESCRIPTION "${PROJECT_DESCRIPTION}" )

message( STATUS "Project '${PROJECT_NAME}', package '${package_name}' version: '${package_version}'")

# Toplevel or subproject:

if ( CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME )
    set( expected_IS_TOPLEVEL_PROJECT TRUE )
else()
    set( expected_IS_TOPLEVEL_PROJECT FALSE )
endif()

# If toplevel project, enable building and performing of tests, disable building of examples:

option( EXPECTED_LITE_OPT_BUILD_TESTS    "Build and perform expected-lite tests" ${expected_IS_TOPLEVEL_PROJECT} )
option( EXPECTED_LITE_OPT_BUILD_EXAMPLES "Build expected-lite examples" OFF )
set(    EXPEXTED_P0323R  "99" STRING     "Specify proposal revision compatibility (99: latest)" )

option( EXPECTED_LITE_OPT_SELECT_STD     "Select std::expected"    OFF )
option( EXPECTED_LITE_OPT_SELECT_NONSTD  "Select nonstd::expected" OFF )

# If requested, build and perform tests, build examples:

if ( EXPECTED_LITE_OPT_BUILD_TESTS )
    enable_testing()
    add_subdirectory( test )
endif()

if ( EXPECTED_LITE_OPT_BUILD_EXAMPLES )
    add_subdirectory( example )
endif()

#
# Interface, installation and packaging
#

# CMake helpers:

include( GNUInstallDirs )
include( CMakePackageConfigHelpers )

# Interface library:

add_library(
    ${package_name} INTERFACE )

add_library(
    ${package_nspace}::${package_name} ALIAS ${package_name} )

target_include_directories(
    ${package_name}
    INTERFACE
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>" )

# Package configuration:
# Note: package_name and package_target are used in package_config_in

set( package_folder         "${package_name}" )
set( package_target         "${package_name}-targets" )
set( package_config         "${package_name}-config.cmake" )
set( package_config_in      "${package_name}-config.cmake.in" )
set( package_config_version "${package_name}-config-version.cmake" )

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${package_config_in}"
    "${CMAKE_CURRENT_BINARY_DIR}/${package_config}"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${package_folder}"
)

configure_file(
   "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${package_config_version}.in"
   "${CMAKE_CURRENT_BINARY_DIR}/${package_config_version}" @ONLY
)

# Installation:

install(
    TARGETS      ${package_name}
    EXPORT       ${package_target}
#   INCLUDES DESTINATION "${...}"  # already set via target_include_directories()
)

install(
    EXPORT       ${package_target}
    NAMESPACE    ${package_nspace}::
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${package_folder}"
)

install(
    FILES       "${CMAKE_CURRENT_BINARY_DIR}/${package_config}"
                "${CMAKE_CURRENT_BINARY_DIR}/${package_config_version}"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${package_folder}"
)

install(
    DIRECTORY   "include/"
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

export(
    EXPORT       ${package_target}
    NAMESPACE    ${package_nspace}::
    FILE         "${CMAKE_CURRENT_BINARY_DIR}/${package_name}-targets.cmake"
)

include(CPack)

# end of file

# ---------------------------------------------------------------
# Programmer(s): Radu Serban, David J. Gardner, Cody J. Balos,
#                and Slaven Peles @ LLNL
# ---------------------------------------------------------------
# SUNDIALS Copyright Start
# Copyright (c) 2002-2025, Lawrence Livermore National Security
# and Southern Methodist University.
# All rights reserved.
#
# See the top-level LICENSE and NOTICE files for details.
#
# SPDX-License-Identifier: BSD-3-Clause
# SUNDIALS Copyright End
# ---------------------------------------------------------------
# Top level CMakeLists.txt for SUNDIALS (for cmake build system)
# ---------------------------------------------------------------

# ===============================================================
# Initial setup.
# ===============================================================

cmake_minimum_required(VERSION 3.18)

# Address DOWNLOAD_EXTRACT_TIMESTAMP warning with CMake 3.24+
if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.24")
  cmake_policy(SET CMP0135 NEW)
endif()

# Project SUNDIALS (initially only C supported) sets PROJECT_SOURCE_DIR and
# PROJECT_BINARY_DIR variables.
project(SUNDIALS C)

# Specify the location of additional CMAKE modules
set(CMAKE_MODULE_PATH
    ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake
    ${PROJECT_SOURCE_DIR}/cmake/macros ${PROJECT_SOURCE_DIR}/cmake/tpl)

# MACRO definitions
include(SundialsCMakeMacros)
include(CMakePrintHelpers)
include(CheckCSourceCompiles)
include(FindPackageHandleStandardArgs)

# Set some variables with info on the SUNDIALS project
set(PACKAGE_BUGREPORT "sundials-users@llnl.gov")
set(PACKAGE_NAME "SUNDIALS")
set(PACKAGE_STRING "SUNDIALS 7.4.0")
set(PACKAGE_TARNAME "sundials")

# Set SUNDIALS version numbers
sundials_git_version() # sets SUNDIALS_GIT_VERSION
message(STATUS "SUNDIALS_GIT_VERSION: ${SUNDIALS_GIT_VERSION}")

# (use "" for the version label if none is needed)
set(PACKAGE_VERSION_MAJOR "7")
set(PACKAGE_VERSION_MINOR "4")
set(PACKAGE_VERSION_PATCH "0")
set(PACKAGE_VERSION_LABEL "")

if(PACKAGE_VERSION_LABEL)
  set(PACKAGE_VERSION
      "${PACKAGE_VERSION_MAJOR}.${PACKAGE_VERSION_MINOR}.${PACKAGE_VERSION_PATCH}-${PACKAGE_VERSION_LABEL}"
  )
else()
  set(PACKAGE_VERSION
      "${PACKAGE_VERSION_MAJOR}.${PACKAGE_VERSION_MINOR}.${PACKAGE_VERSION_PATCH}"
  )
endif()

# Specify the VERSION and SOVERSION for shared libraries

set(arkodelib_VERSION "6.4.0")
set(arkodelib_SOVERSION "6")

set(cvodelib_VERSION "7.4.0")
set(cvodelib_SOVERSION "7")

set(cvodeslib_VERSION "7.4.0")
set(cvodeslib_SOVERSION "7")

set(idalib_VERSION "7.4.0")
set(idalib_SOVERSION "7")

set(idaslib_VERSION "6.4.0")
set(idaslib_SOVERSION "6")

set(kinsollib_VERSION "7.4.0")
set(kinsollib_SOVERSION "7")

set(cpodeslib_VERSION "0.0.0")
set(cpodeslib_SOVERSION "0")

set(nveclib_VERSION "7.4.0")
set(nveclib_SOVERSION "7")

set(sunmatrixlib_VERSION "5.4.0")
set(sunmatrixlib_SOVERSION "5")

set(sunlinsollib_VERSION "5.4.0")
set(sunlinsollib_SOVERSION "5")

set(sunnonlinsollib_VERSION "4.4.0")
set(sunnonlinsollib_SOVERSION "4")

set(sundialslib_VERSION
    "${PACKAGE_VERSION_MAJOR}.${PACKAGE_VERSION_MINOR}.${PACKAGE_VERSION_PATCH}"
)
set(sundialslib_SOVERSION "${PACKAGE_VERSION_MAJOR}")

# ===============================================================
# Initial Setup
# ===============================================================

# Prohibit in-source build
if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
  message(FATAL_ERROR "In-source build prohibited.")
endif()

# Organize targets into folders when using an IDE
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Get correct build paths automatically, but expose LIBDIR and INCLUDEDIR as a
# regular cache variable so that a user can more easily see what they were set
# to by GNUInstallDirs.
include(GNUInstallDirs)
mark_as_advanced(CLEAR CMAKE_INSTALL_LIBDIR)
mark_as_advanced(CLEAR CMAKE_INSTALL_INCLUDEDIR)

# Suffixes to use for static and shared targets.
set(_STATIC_LIB_SUFFIX
    "_static"
    CACHE INTERNAL "" FORCE)
set(_SHARED_LIB_SUFFIX
    "_shared"
    CACHE INTERNAL "" FORCE)

# A list of all the alias targets created.
set(_SUNDIALS_ALIAS_TARGETS
    ""
    CACHE INTERNAL "" FORCE)

# We default to release builds
set(_DEFAULT_CMAKE_BUILD_TYPE RelWithDebInfo)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(
    STATUS
      "Building SUNDIALS in '${_DEFAULT_CMAKE_BUILD_TYPE}' mode as CMAKE_BUILD_TYPE was not specified."
  )
  set(CMAKE_BUILD_TYPE
      "${_DEFAULT_CMAKE_BUILD_TYPE}"
      CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
                                               "MinSizeRel" "RelWithDebInfo")
else()
  message(STATUS "Building SUNDIALS in '${CMAKE_BUILD_TYPE}' mode.")
endif()

# ===============================================================
# Build options to be processed first
# ===============================================================

include(SundialsDeprecated)
include(SundialsBuildOptionsPre)

# ===============================================================
# Options for external packages
# ===============================================================

include(SundialsTPLOptions)

# ===============================================================
# Options for examples
# ===============================================================

include(SundialsExampleOptions)

# ===============================================================
# Setup compilers
# ===============================================================

include(SundialsSetupCompilers)

# ===============================================================
# Setup third-party libraries
# ===============================================================

include(SundialsSetupTPLs)

# ===============================================================
# Build options to be processed last
# ===============================================================

include(SundialsBuildOptionsPost)

# ===============================================================
# At this point all the configuration options are set. Setup the
# sundials_config.h.
# ===============================================================

include(SundialsSetupConfig)

# ===============================================================
# Setup testing
# ===============================================================

include(SundialsSetupTesting)

# ===============================================================
# Add src and tests; optionally add examples and unit tests.
# ===============================================================

# Add selected packages and modules to the build
add_subdirectory(src)

# Add selected examples to the build
if(_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

# Add benchmarks to the build
if(BUILD_BENCHMARKS)
  include(SundialsSetupBenchmarks)
  add_subdirectory(benchmarks)
endif()

# Add unit tests to the build
if(SUNDIALS_TEST_ENABLE_UNIT_TESTS)
  add_subdirectory(test/unit_tests)
endif()

if(SUNDIALS_ENABLE_EXTERNAL_ADDONS)
  add_subdirectory(external)
endif()

# ===============================================================
# Install configuration header files and license file.
# ===============================================================

# install sundials_export header file
install(FILES "${PROJECT_BINARY_DIR}/include/sundials/sundials_export.h"
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/sundials")

# install configured header file
install(FILES "${PROJECT_BINARY_DIR}/include/sundials/sundials_config.h"
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/sundials")

# install shared Fortran 2003 modules
if(BUILD_FORTRAN_MODULE_INTERFACE)
  # While the .mod files get generated for static and shared libraries, they are
  # identical. So only install one set of the .mod files.
  if(BUILD_STATIC_LIBS)
    install(DIRECTORY ${CMAKE_Fortran_MODULE_DIRECTORY}_STATIC/
            DESTINATION ${Fortran_INSTALL_MODDIR})
  else()
    install(DIRECTORY ${CMAKE_Fortran_MODULE_DIRECTORY}_SHARED/
            DESTINATION ${Fortran_INSTALL_MODDIR})
  endif()
endif()

# install license and notice files
install(FILES "${PROJECT_SOURCE_DIR}/LICENSE"
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/sundials")
install(FILES "${PROJECT_SOURCE_DIR}/NOTICE"
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/sundials")

# create package version file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  SUNDIALSConfigVersion.cmake
  VERSION ${PACKAGE_VERSION}
  COMPATIBILITY AnyNewerVersion)

# install targets
install(
  EXPORT sundials-targets
  FILE SUNDIALSTargets.cmake
  NAMESPACE SUNDIALS::
  DESTINATION "${SUNDIALS_INSTALL_CMAKEDIR}")

# install SUNDIALSConfig.cmake
configure_package_config_file(
  "${PROJECT_SOURCE_DIR}/cmake/SUNDIALSConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/SUNDIALSConfig.cmake"
  INSTALL_DESTINATION "${SUNDIALS_INSTALL_CMAKEDIR}")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/SUNDIALSConfig.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/SUNDIALSConfigVersion.cmake"
        DESTINATION "${SUNDIALS_INSTALL_CMAKEDIR}")

# Export targets so build directory can be used directly
export(
  EXPORT sundials-targets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/SUNDIALSTargets.cmake"
  NAMESPACE SUNDIALS::)

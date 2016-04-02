################################################################################
# Project Properties
################################################################################
cmake_minimum_required (VERSION 3.1)
set (PROJECT "proj5")
project (${PROJECT})
set (APPLICATION_NAME "${PROJECT}")
set (APPLICATION_CODENAME "${PROJECT}")
set (APPLICATION_COPYRIGHT_YEARS 2015)
set (APPLICATION_VERSION_MAJOR 0)
set (APPLICATION_VERSION_MINOR 0)
set (APPLICATION_VERSION_PATCH 0)
set (APPLICATION_VERSION_TYPE SNAPSHOT)
set (APPLICATION_VERSION_STRING "${APPLICATION_VERSION_MAJOR}.${APPLICATION_VERSION_MINOR}.${APPLICATION_VERSION_PATCH}-${APPLICATION_VERSION_TYPE}")

################################################################################
# Compiler Options
################################################################################
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-std=c++0x" COMPILER_SUPPORTS_CXX11)
if(COMPILER_SUPPORTS_CXX11)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
  message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++0x support. Please use a different C++ compiler.")
endif()

################################################################################
# Debugging Options
################################################################################
set (CMAKE_VERBOSE_MAKEFILE 0) # Use 1 for debugging, 0 for release
set (BUILD_SHARED_LIBS 0)

################################################################################
# Project Search Paths
################################################################################
# Find includes in corresponding build directories
set (CMAKE_INCLUDE_CURRENT_DIR ON)
set (CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake_modules" ${CMAKE_MODULE_PATH})
list (APPEND CMAKE_PREFIX_PATH "${MAINFOLDER}")
include_directories(${GTEST_INCLUDE_DIRS} ${PROJECT_SOURCE_DIR}/src)

################################################################################
# Project Output Paths
################################################################################
set (MAINFOLDER ${PROJECT_SOURCE_DIR})
set (EXECUTABLE_OUTPUT_PATH "${MAINFOLDER}/bin")
set (LIBRARY_OUTPUT_PATH "${MAINFOLDER}/lib")

################################################################################
# Configure Files
################################################################################
file (GLOB_RECURSE CONFIGINPUTS1 src/*.in.h.cmake)
file (GLOB_RECURSE CONFIGINPUTS2 src/*.h.in.cmake)
file (GLOB_RECURSE CONFIGINPUTS3 test/*.in.h.cmake)
file (GLOB_RECURSE CONFIGINPUTS4 test/*.h.in.cmake)
list (APPEND CONFIGINPUTS "${CONFIGINPUTS1}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS2}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS3}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS4}")
foreach (CONFIGINPUT ${CONFIGINPUTS})
  string (REPLACE ".in.h.cmake" ".h" CONFIGOUTPUT1 "${CONFIGINPUT}")
  string (REPLACE ".h.in.cmake" ".h" CONFIGOUTPUT2 "${CONFIGOUTPUT1}")
  configure_file ("${CONFIGINPUT}" "${CONFIGOUTPUT2}")
endforeach (CONFIGINPUT ${CONFIGINPUTS})

################################################################################
# Find needed packages
################################################################################

################################################################################
# Find testing packages
################################################################################
find_package(GTest REQUIRED)

################################################################################
# Add Documentation Targets
################################################################################
include (DocumentationTargets)

################################################################################
# Build Targets
################################################################################
add_subdirectory(src)

if(GTEST_FOUND)
  add_subdirectory(test)
endif()

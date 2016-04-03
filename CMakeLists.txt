################################################################################
# Project Properties
################################################################################
cmake_minimum_required (VERSION 2.8.8)
set (PROJECT_NAME "proj6")
set (APPLICATION_COPYRIGHT_YEARS 2016)
set (APPLICATION_VERSION_MAJOR 0)
set (APPLICATION_VERSION_MINOR 0)
set (APPLICATION_VERSION_PATCH 0)
set (APPLICATION_VERSION_TYPE SNAPSHOT)

set (BUILD_SHARED_LIBS 1)

set (CMAKE_VERBOSE_MAKEFILE 0) # Use 1 for debugging, 0 for release

################################################################################
# Configure version and naming information
################################################################################
project (${PROJECT_NAME} C CXX)

set (APPLICATION_NAME "${PROJECT_NAME}")
set (APPLICATION_CODENAME "${PROJECT_NAME}")
set (APPLICATION_VERSION_STRING "${APPLICATION_VERSION_MAJOR}.${APPLICATION_VERSION_MINOR}.${APPLICATION_VERSION_PATCH}-${APPLICATION_VERSION_TYPE}")

file (GLOB_RECURSE CONFIGINPUTS1 include/*.in.h.cmake)
file (GLOB_RECURSE CONFIGINPUTS2 include/*.h.in.cmake)
list (APPEND CONFIGINPUTS "${CONFIGINPUTS1}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS2}")
foreach (CONFIGINPUT ${CONFIGINPUTS})
  string (REPLACE ".in.h.cmake" ".h" CONFIGOUTPUT1 "${CONFIGINPUT}")
  string (REPLACE ".h.in.cmake" ".h" CONFIGOUTPUT2 "${CONFIGOUTPUT1}")
  configure_file ("${CONFIGINPUT}" "${CONFIGOUTPUT2}")
endforeach (CONFIGINPUT ${CONFIGINPUTS})

################################################################################
# Compiler Options
################################################################################
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_CXX11)
if(COMPILER_SUPPORTS_CXX11)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
else()
  message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

if (CMAKE_COMPILER_IS_GNUCXX)
  add_definitions(-Wall -ansi -Wno-deprecated -pthread)
endif()

if (MSVC)
  #vc 2012 fix for vararg templates
  set(MSVC_COMPILER_DEFS "-D_VARIADIC_MAX=10")
endif()

################################################################################
# Misc options
################################################################################
# Enable ExternalProject CMake module
include (ExternalProject)

# Threads are needed by google test
find_package (Threads REQUIRED)

################################################################################
# Project Search Paths
################################################################################
set (CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake_modules" ${CMAKE_MODULE_PATH})
set (COMMON_INCLUDES ${PROJECT_SOURCE_DIR}/include)
include_directories(
  ${COMMON_INCLUDES}
  )

################################################################################
# Find needed depedencies
################################################################################

################################################################################
# Setup Gtest
################################################################################
ExternalProject_Add (googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  CMAKE_ARGS -DCMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG:PATH=DebugLibs
  -DCMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE:PATH=ReleaseLibs
  -DCMAKE_CXX_FLAGS=${MSVC_COMPILER_DEFS}
  -Dgtest_force_shared_crt=ON
  -DBUILD_GTEST=ON
  PREFIX "${CMAKE_CURRENT_BINARY_DIR}"
  # Disable install step
  INSTALL_COMMAND ""
  )

# Specify include dir
ExternalProject_Get_Property (googletest source_dir)
set (GTEST_INCLUDE_DIRS ${source_dir}/googletest/include)

# Specify MainTest's link libraries
ExternalProject_Get_Property (googletest binary_dir)
set (GTEST_LIBS_DIR ${binary_dir}/googlemock/gtest)

################################################################################
# Add Documentation Targets
################################################################################
include (DocumentationTargets)

################################################################################
# Build Targets
################################################################################
file (GLOB_RECURSE SRC_FILES
  ${PROJECT_SOURCE_DIR}/src/*.cpp
  ${PROJECT_SOURCE_DIR}/src/*.cxx
  ${PROJECT_SOURCE_DIR}/src/*.cc
  ${PROJECT_SOURCE_DIR}/src/*.C
  ${PROJECT_SOURCE_DIR}/src/*.c
  )

add_executable(${PROJECT_NAME} ${COMMON_INCLUDES} ${PROJECT_SOURCE_DIR}/main.cpp ${SRC_FILES})
target_link_libraries (${PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT})

################################################################################
# Test Targets
################################################################################
set (PROJECT_TEST_NAME ${PROJECT_NAME}_test)
add_definitions(${MSVC_COMPILER_DEFS})
enable_testing()

include_directories(
  ${COMMON_INCLUDES}
  ${GTEST_INCLUDE_DIRS}
  )

file (GLOB TEST_SRC_FILES
  ${PROJECT_SOURCE_DIR}/src/*.cpp
  ${PROJECT_SOURCE_DIR}/src/*.cxx
  ${PROJECT_SOURCE_DIR}/src/*.cc
  ${PROJECT_SOURCE_DIR}/src/*.C
  ${PROJECT_SOURCE_DIR}/src/*.c
  ${PROJECT_SOURCE_DIR}/test/*.cpp
  ${PROJECT_SOURCE_DIR}/test/*.cxx
  ${PROJECT_SOURCE_DIR}/test/*.cc
  ${PROJECT_SOURCE_DIR}/test/*.C
  ${PROJECT_SOURCE_DIR}/test/*.c
  ${PROJECT_SOURCE_DIR}/test/*.h
  ${PROJECT_SOURCE_DIR}/test/*.hpp
  )

add_executable (${PROJECT_TEST_NAME} ${COMMON_INCLUDES} ${TEST_SRC_FILES})
add_dependencies (${PROJECT_TEST_NAME} googletest)

if(WIN32)
  target_link_libraries(${PROJECT_TEST_NAME}
    debug ${GTEST_LIBS_DIR}/DebugLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${CMAKE_FIND_LIBRARY_SUFFIXES}
    optimized ${GTEST_LIBS_DIR}/ReleaseLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${CMAKE_FIND_LIBRARY_SUFFIXES}
    )
  target_link_libraries(${PROJECT_TEST_NAME}
    debug ${GTEST_LIBS_DIR}/DebugLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main${CMAKE_FIND_LIBRARY_SUFFIXES}
    optimized ${GTEST_LIBS_DIR}/ReleaseLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main${CMAKE_FIND_LIBRARY_SUFFIXES}
    )
else()
  target_link_libraries(${PROJECT_TEST_NAME}
    ${GTEST_LIBS_DIR}/libgtest.a
    ${GTEST_LIBS_DIR}/libgtest_main.a
    )
endif()

target_link_libraries(${PROJECT_TEST_NAME} ${CMAKE_THREAD_LIBS_INIT})

################################################################################
# Installation
################################################################################
# Project Output Paths
set (EXECUTABLE_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/bin")

install (TARGETS ${PROJECT_NAME} DESTINATION bin)
install (TARGETS ${TEST_BIN} DESTINATION bin)

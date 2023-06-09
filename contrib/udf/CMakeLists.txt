# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

cmake_minimum_required(VERSION 3.19.2)
 
# set CMAKE_C_COMPILER, this must set before project command
if (DEFINED ENV{DORIS_GCC_HOME})
    set(CMAKE_C_COMPILER "$ENV{DORIS_GCC_HOME}/bin/gcc")
    set(CMAKE_CXX_COMPILER "$ENV{DORIS_GCC_HOME}/bin/g++")
    set(GCC_HOME $ENV{DORIS_GCC_HOME})
else()
    message(FATAL_ERROR "DORIS_GCC_HOME environment variable is not set")
endif()

project(doris_udf)

# set CMAKE_BUILD_TYPE
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE RELEASE)
endif()

string(TOUPPER ${CMAKE_BUILD_TYPE} CMAKE_BUILD_TYPE)
message(STATUS "Build type is ${CMAKE_BUILD_TYPE}")

set(BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(ENV{DORIS_HOME} "${BASE_DIR}/../../")
set(THIRDPARTY_DIR "$ENV{DORIS_THIRDPARTY}/installed/")
set(BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}")
set(SRC_DIR "${BASE_DIR}/src/")
set(OUTPUT_DIR "${BASE_DIR}/output")

# Check gcc
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS "7.3.0")
        message(FATAL_ERROR "Need GCC version at least 7.3.0")
    endif()

    if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER "7.3.0")
        message(STATUS "GCC version is greater than 7.3.0, disable -Werror. Be careful with compile warnings.")
    else()
        #   -Werror: compile warnings should be errors when using the toolchain compiler.
        set(CXX_GCC_FLAGS "${CXX_GCC_FLAGS} -Werror")
    endif()
elseif (NOT APPLE)
    message(FATAL_ERROR "Compiler should be GNU")
endif()

# Just for clang-tidy:  -Wno-expansion-to-defined -Wno-deprecated-declaration
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -g -ggdb -std=c++11 -Wall -Werror -Wno-unused-variable -Wno-expansion-to-defined -Wno-deprecated-declarations -O3")

if (CMAKE_GENERATOR STREQUAL "Ninja" AND NOT DISABLE_COLORED_BUILD)
    # Turn on colored output. https://github.com/ninja-build/ninja/wiki/FAQ
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color=always")
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fdiagnostics-color=always")
endif ()

message(STATUS "Compiler Flags: ${CMAKE_CXX_FLAGS}")

# Include udf
include_directories($ENV{DORIS_HOME}/output/udf/include)

# Set all libraries
add_library(udf STATIC IMPORTED)
set_target_properties(udf PROPERTIES IMPORTED_LOCATION $ENV{DORIS_HOME}/output/udf/lib/libDorisUdf.a)

# Add the subdirector of new UDF in here
add_subdirectory(${SRC_DIR}/udf_samples)

install(DIRECTORY DESTINATION ${OUTPUT_DIR})

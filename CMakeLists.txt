# ================================================================================================ #
#                                                                                                  #
#                                        Project: osmpbf                                           #
#                                                                                                  #
# ================================================================================================ #
#                                                                                                  #
#                                            LICENSE                                               #
#                                                                                                  #
# ------------------------------------------------------------------------------------------------ #
#                                                                                                  #
# MIT License                                                                                      #
#                                                                                                  #
# Copyright (c) 2018 Michael Herwig                                                                #
#                                                                                                  #
# Permission is hereby granted, free of charge, to any person obtaining a copy of this software    #
# and associated documentation files (the "Software"), to deal in the Software without             #
# restriction, including without limitation the rights to use, copy, modify, merge, publish,       #
# distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the    #
# Software is furnished to do so, subject to the following conditions:                             #
#                                                                                                  #
# The above copyright notice and this permission notice shall be included in all copies or         #
# substantial portions of the Software.                                                            #
#                                                                                                  #
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING    #
# BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND       #
# NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,     #
# DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,   #
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.          #
#                                                                                                  #
# ================================================================================================ #

cmake_minimum_required(VERSION 3.0.0)

# ------------------------------------------------------------------------------------------------ #
# hunter                                                                                           #
# ------------------------------------------------------------------------------------------------ #

include(cmake/HunterGate.cmake)
HunterGate(
  URL "https://github.com/ruslo/hunter/archive/v0.20.68.tar.gz"
  SHA1 "8bbef873746f3e47595550751c052098aa8d202e"
)

# ------------------------------------------------------------------------------------------------ #
# Project Definition                                                                               #
# ------------------------------------------------------------------------------------------------ #

project(osmpbf
  VERSION
    0.0.0

  DESCRIPTION
    "A parser for reading osm pbf files"

  LANGUAGES
    CXX
    C
)

# ------------------------------------------------------------------------------------------------ #
# Options                                                                                          #
# ------------------------------------------------------------------------------------------------ #

option(OSMPBF_BUILD_TESTS "Enables integration of test suites" OFF)
option(OSMPBF_BUILD_BENCHMARKS "Enables integration of benchmarks" OFF)

# ------------------------------------------------------------------------------------------------ #
# Dependencies                                                                                     #
# ------------------------------------------------------------------------------------------------ #

hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED)

hunter_add_package(Protobuf)
find_package(Protobuf CONFIG REQUIRED)

hunter_add_package(Leathers)
find_package(Leathers CONFIG REQUIRED)

if(OSMPBF_BUILD_TESTS)
  hunter_add_package(GTest)
  find_package(GMock CONFIG REQUIRED)
endif()

if(OSMPBF_BUILD_BENCHMARKS)
  hunter_add_package(benchmark)
  find_package(benchmark CONFIG REQUIRED)
endif()

# ------------------------------------------------------------------------------------------------ #
# Compiler Configuration                                                                           #
# ------------------------------------------------------------------------------------------------ #

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------------------------------------ #
# Protobuf                                                                                         #
# ------------------------------------------------------------------------------------------------ #

set(PBF_SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/protobuf)

file(MAKE_DIRECTORY ${PBF_SOURCE_DIR})

set(PBF_DEFINITION_FILES
  protobuf/fileformat.proto
  protobuf/osmformat.proto
)
set(PBF_GENERATED_FILES
  ${PBF_SOURCE_DIR}/protobuf/fileformat.pb.h
  ${PBF_SOURCE_DIR}/protobuf/fileformat.pb.cc
  ${PBF_SOURCE_DIR}/protobuf/osmformat.pb.h
  ${PBF_SOURCE_DIR}/protobuf/osmformat.pb.cc
)

add_custom_command(OUTPUT ${PBF_GENERATED_FILES}
  COMMENT "Generating protobuf implementation files..."

  COMMAND protobuf::protoc
    -I=${CMAKE_CURRENT_SOURCE_DIR}
    --cpp_out=${PBF_SOURCE_DIR}
    ${PBF_DEFINITION_FILES}

  DEPENDS ${PBF_DEFINITION_FILES}
)

add_custom_target(osmpbf-protobuf
  DEPENDS ${PBF_GENERATED_FILES}
  SOURCES ${PBF_DEFINITION_FILES}
)

# ------------------------------------------------------------------------------------------------ #
# Library: osmpbf                                                                                  #
# ------------------------------------------------------------------------------------------------ #

add_library(osmpbf STATIC
  source/fileformat.cpp
  source/osmformat.cpp
)

target_include_directories(osmpbf
  PRIVATE
    ${PBF_SOURCE_DIR}
)

add_dependencies(osmpbf osmpbf-protobuf)

target_link_libraries(osmpbf
  PRIVATE
    Leathers::leathers
    ZLIB::zlib
    protobuf::libprotobuf
)

# ------------------------------------------------------------------------------------------------ #
# Benchmarks                                                                                       #
# ------------------------------------------------------------------------------------------------ #

if(OSMPBF_BUILD_BENCHMARKS)
  add_executable(osmpbf-benchmarks
    benchmarks/main.cpp
    benchmarks/b_test.cpp
  )

  target_link_libraries(osmpbf-benchmarks
    PRIVATE
      benchmark::benchmark
      osmpbf
  )
endif()

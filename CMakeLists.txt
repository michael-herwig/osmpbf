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
option(OSMPBF_BUILD_DOCS "Enables integration of doxygen documentation generation" OFF)

option(OSMPBF_ENABLE_CPPCHECK "Enables static analyzer 'cppcheck'" OFF)

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

if(MSVC)
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /ignore:4099")
  string(REGEX REPLACE "/W[0-4]" "" CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /WX")
endif()

# ------------------------------------------------------------------------------------------------ #
# cppcheck                                                                                         #
# ------------------------------------------------------------------------------------------------ #

if(OSMPBF_ENABLE_CPPCHECK)
  find_program(CMAKE_CXX_CPPCHECK NAMES cppcheck)
  if(NOT CMAKE_CXX_CPPCHECK)
    message(FATAL_ERROR "Could not find cppcheck.")
  endif()
  list(APPEND CMAKE_CXX_CPPCHECK
    --enable=all
    --template=vs
    --xml
  )
  if(WIN32)
    list(APPEND CMAKE_CXX_CPPCHECK --platform=windows)
  elseif(UNIX)
    list(APPEND CMAKE_CXX_CPPCHECK --platform=linux)
  endif()
endif()

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

set(SOURCE_FILES
  include/osmpbf/osmpbf
  include/osmpbf/osmpbf.hpp

  source/fileformat.cpp
  source/osmformat.cpp
)

add_library(osmpbf STATIC ${SOURCE_FILES})

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
# Tests                                                                                            #
# ------------------------------------------------------------------------------------------------ #

if(OSMPBF_BUILD_TESTS)
  add_executable(osmpbf-tests
    tests/t_test.cpp
  )

  target_link_libraries(osmpbf-tests
    PRIVATE
      Leathers::leathers
      GMock::main
      osmpbf
  )

  enable_testing()
  add_test(osmpbf-test COMMAND osmpbf-test)
endif()

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
      Leathers::leathers
      benchmark::benchmark
      osmpbf
  )
endif()

# ------------------------------------------------------------------------------------------------ #
# Doxygen                                                                                          #
# ------------------------------------------------------------------------------------------------ #

if(OSMPBF_BUILD_DOCS)
  find_package(Doxygen REQUIRED)
  doxygen_add_docs(osmpbf-documentation
    ${SOURCE_FILES}
  )
endif()

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

project(osmpbf CXX C)

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

if(OSMPBF_BUILD_TESTS)
  hunter_add_package(GTest)
  find_package(GMock CONFIG REQUIRED)
endif()

if(OSMPBF_BUILD_BENCHMARKS)
  hunter_add_package(benchmark)
  find_package(benchmark CONFIG REQUIRED)
endif()

# ------------------------------------------------------------------------------------------------ #
# Library: osmpbf                                                                                  #
# ------------------------------------------------------------------------------------------------ #

# target_link_libraries(... ZLIB::zlib protobuf::libprotobuf)

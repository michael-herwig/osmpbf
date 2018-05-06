cmake_minimum_required(VERSION 3.0.0)

include(cmake/HunterGate.cmake)
HunterGate(
  URL "https://github.com/ruslo/hunter/archive/v0.20.68.tar.gz"
  SHA1 "8bbef873746f3e47595550751c052098aa8d202e"
)

project(osmpbf CXX C)

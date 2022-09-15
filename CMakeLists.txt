# CMake project definition for Snap.
#
# The upstream provides Makefiles for building the project. However, for
# macposts we would like to have a CMake project for easy integration.

cmake_minimum_required(VERSION 3.10)

project(Snap)
set(CMAKE_CXX_STANDARD 98)
find_package(OpenMP)

add_library(snap STATIC snap-core/Snap.cpp)

target_include_directories(snap
  INTERFACE snap-core
  PUBLIC glib-core
  )

if (OpenMP_CXX_FOUND)
  target_link_libraries(snap PRIVATE OpenMP::OpenMP_CXX)
else()
  # NOTE: From Makefile.config (for Darwin build). Not sure what it does.
  target_compile_definitions(snap PRIVATE NOMP)
endif()

if (UNIX AND NOT APPLE)
  target_link_libraries(snap PRIVATE rt)
endif()

# MSVC with its defaults bails out on Snap.o (on GitHub Actions
# "windows-latest").
if (MSVC)
  target_compile_options(snap PRIVATE /bigobj)
endif()

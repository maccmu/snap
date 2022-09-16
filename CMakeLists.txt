# CMake project definition for Snap.
#
# The upstream provides Makefiles for building the project. However, for
# macposts we would like to have a CMake project for easy integration.

cmake_minimum_required(VERSION 3.14)

project(Snap)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)
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

# Tests

option(SNAP_BUILD_TESTS "Build all Snap tests." OFF)

if (SNAP_BUILD_TESTS)
  include(FetchContent)
  FetchContent_Declare(googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        release-1.11.0
    )
  set(gtest_force_shared_crt ON CACHE BOOL "Force shared CRT" FORCE)
  FetchContent_MakeAvailable(googletest)

  enable_testing()
  add_executable(run-all-tests
    test/run-all-tests.cpp
    test/test-helper.cpp
    test/test-TUNGraph.cpp
    test/test-TNGraph.cpp
    test/test-TNEGraph.cpp
    test/test-TNEANet.cpp
    test/test-TNodeNet.cpp
    test/test-TNodeEDatNet.cpp
    test/test-TNodeEdgeNet.cpp
    test/test-TTable.cpp
    test/test-TUndirNet.cpp
    test/test-TDirNet.cpp
    test/test-TMMNet.cpp
    test/test-TModeNet.cpp
    test/test-TCrossNet.cpp
    test/test-multimodal.cpp
    test/test-subgraph.cpp
    test/test-TStrPool.cpp
    test/test-TSysTm.cpp
    test/test-ggen.cpp
    test/test-gio.cpp
    test/test-gviz.cpp
    test/test-cncom.cpp
    test/test-bfsdfs.cpp
    test/test-alg.cpp
    test/test-triad.cpp
    test/test-THash.cpp
    test/test-THashSet.cpp
    test/test-TAttr.cpp
    test/test-flow.cpp
    test/test-randwalk.cpp
    test/test-priority-queue.cpp
    test/test-sim.cpp
    )
  target_link_libraries(run-all-tests PRIVATE snap gtest_main)

  include(GoogleTest)
  gtest_discover_tests(run-all-tests)
endif()

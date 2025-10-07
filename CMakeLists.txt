cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

set(PROJECT_OUTPUT_FOLDER "${CMAKE_CURRENT_SOURCE_DIR}/../test")
set(CMAKE_CONFIGURATION_TYPES "Release;Debug" CACHE STRING "Release;Debug")

project(pc_error)

# --- PCL ---
# Keep the components you use; IO is crucial for PLY/PCD symbols
find_package(PCL 1.8 REQUIRED COMPONENTS common octree io search features)

# --- OpenMP (optional) ---
find_package(OpenMP)
if (OPENMP_FOUND)
  message(STATUS "OpenMP found.")
  set (CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# --- Boost program_options (REQUIRED!) ---
set(Boost_USE_STATIC_LIBS        OFF)
set(Boost_USE_MULTITHREADED      ON)
set(Boost_USE_STATIC_RUNTIME     OFF)
find_package(Boost 1.61.0 REQUIRED COMPONENTS program_options)

include_directories(
  "${CMAKE_CURRENT_SOURCE_DIR}/./"
  ${PCL_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

# C++11 for both C and C++ compilers (CMake 2.8 style)
set (CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -std=c++11")
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

# Sources
file(GLOB_RECURSE PROJ_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/*.h*)
file(GLOB SOURCE               ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)

# Do NOT strip libs out of PCL_LIBRARIES; some are needed for IO
# (The previous REMOVE_ITEM could drop required deps and cause link errors.)
message(STATUS "PCL_LIBRARIES=${PCL_LIBRARIES}")

# Target
if (MSVC)
  message(STATUS "TO BE TESTED")
  add_executable(pc_error ${SOURCE})
else()
  list(REMOVE_ITEM SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/stdafx.cpp")
  add_executable(pc_error ${SOURCE})
endif()

# Link: PCL + Boost::program_options
target_link_libraries(pc_error
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES}
)

# No need to manually find pcl_io_ply / pcl_io_pcd when PCL IO is linked.
# If your PCL splits them and you still need them, you can re-enable the block below.
# find_library(PCL_IO_PLY_LIB pcl_io_ply HINTS ${PCL_LIBRARY_DIRS})
# find_library(PCL_IO_PCD_LIB pcl_io_pcd HINTS ${PCL_LIBRARY_DIRS})
# if (PCL_IO_PLY_LIB)
#   target_link_libraries(pc_error ${PCL_IO_PLY_LIB})
# endif()
# if (PCL_IO_PCD_LIB)
#   target_link_libraries(pc_error ${PCL_IO_PCD_LIB})
# endif()

# Install & output dirs
install(TARGETS pc_error DESTINATION ".")
set_target_properties(pc_error PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_OUTPUT_FOLDER})
set_target_properties(pc_error PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG   ${PROJECT_OUTPUT_FOLDER})
set_target_properties(pc_error PROPERTIES DEBUG_POSTFIX "_d")

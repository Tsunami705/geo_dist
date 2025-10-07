cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

set(PROJECT_OUTPUT_FOLDER "${CMAKE_CURRENT_SOURCE_DIR}/../test")
set(CMAKE_CONFIGURATION_TYPES "Release;Debug" CACHE STRING "Release;Debug")

project(pc_error)

add_definitions(-D_GLIBCXX_USE_CXX11_ABI=0)

# --- PCL (explicit components) ---
find_package(PCL 1.8 REQUIRED COMPONENTS common io octree search features)

# --- OpenMP (optional) ---
find_package(OpenMP)
if (OPENMP_FOUND)
  message(STATUS "OpenMP found.")
  set (CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# --- Threads (pthread) ---
find_package(Threads REQUIRED)

# --- Boost: request all pieces we might need explicitly ---
set(Boost_USE_STATIC_LIBS        OFF)
set(Boost_USE_MULTITHREADED      ON)
set(Boost_USE_STATIC_RUNTIME     OFF)
find_package(Boost 1.61.0 REQUIRED
  COMPONENTS program_options filesystem system iostreams
)

include_directories(
  "${CMAKE_CURRENT_SOURCE_DIR}/./"
  ${PCL_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)
link_directories(${PCL_LIBRARY_DIRS})


# C++11 (CMake 2.8 style)
set (CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -std=c++11")
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

# Sources
file(GLOB_RECURSE PROJ_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/*.h*)
file(GLOB SOURCE               ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
list(REMOVE_ITEM SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/stdafx.cpp")

add_executable(pc_error ${SOURCE})

# --- Link explicitly: PCL IO + COMMON, then the rest, then Boost components, then pthread ---
# Some PCL 1.8 exports don't propagate IO/Common properly via PCL_LIBRARIES, so be explicit first.
target_link_libraries(pc_error
  ${PCL_IO_LIBRARIES}
  ${PCL_COMMON_LIBRARIES}
  ${PCL_LIBRARIES}
  ${Boost_PROGRAM_OPTIONS_LIBRARY}
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  ${Boost_IOSTREAMS_LIBRARY}
  Threads::Threads
)

# Optional: conda’s PCL 1.8 may split extra IO libs; add them if they exist.
find_library(PCL_IO_PLY_LIB pcl_io_ply HINTS ${PCL_LIBRARY_DIRS})
find_library(PCL_IO_PCD_LIB pcl_io_pcd HINTS ${PCL_LIBRARY_DIRS})
if (PCL_IO_PLY_LIB)
  message(STATUS "Found pcl_io_ply: ${PCL_IO_PLY_LIB}")
  target_link_libraries(pc_error ${PCL_IO_PLY_LIB})
endif()
if (PCL_IO_PCD_LIB)
  message(STATUS "Found pcl_io_pcd: ${PCL_IO_PCD_LIB}")
  target_link_libraries(pc_error ${PCL_IO_PCD_LIB})
endif()

# Output / install
install(TARGETS pc_error DESTINATION ".")
set_target_properties(pc_error PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_OUTPUT_FOLDER}
  RUNTIME_OUTPUT_DIRECTORY_DEBUG   ${PROJECT_OUTPUT_FOLDER}
  DEBUG_POSTFIX "_d"
)

# Debug prints so we can see exactly what CMake resolved
message(STATUS "PCL_LIBRARIES               = ${PCL_LIBRARIES}")
message(STATUS "PCL_IO_LIBRARIES            = ${PCL_IO_LIBRARIES}")
message(STATUS "PCL_COMMON_LIBRARIES        = ${PCL_COMMON_LIBRARIES}")
message(STATUS "Boost_LIBRARIES             = ${Boost_LIBRARIES}")
message(STATUS "Boost_PROGRAM_OPTIONS_LIB   = ${Boost_PROGRAM_OPTIONS_LIBRARY}")
message(STATUS "Boost_FILESYSTEM_LIB        = ${Boost_FILESYSTEM_LIBRARY}")
message(STATUS "Boost_SYSTEM_LIB            = ${Boost_SYSTEM_LIBRARY}")
message(STATUS "Boost_IOSTREAMS_LIB         = ${Boost_IOSTREAMS_LIBRARY}")

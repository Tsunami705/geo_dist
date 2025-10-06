cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(pc_error)

set(PROJECT_OUTPUT_FOLDER "${CMAKE_CURRENT_SOURCE_DIR}/../test")
set(CMAKE_CONFIGURATION_TYPES "Release;Debug" CACHE STRING "Release;Debug")

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(PCL 1.8 COMPONENTS common octree io search features REQUIRED)

set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package(Boost 1.61.0 REQUIRED COMPONENTS program_options)

find_package(OpenMP)
if (OpenMP_CXX_FOUND)
  message(STATUS "OpenMP found.")
  target_compile_options(${PROJECT_NAME} INTERFACE ${OpenMP_CXX_FLAGS})
endif()

file(GLOB_RECURSE PROJ_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/*.h*)
file(GLOB SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
list(REMOVE_ITEM SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/stdafx.cpp")

add_executable(pc_error ${SOURCE})

target_include_directories(pc_error PRIVATE
  "${CMAKE_CURRENT_SOURCE_DIR}"
  ${PCL_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)
add_definitions(${PCL_DEFINITIONS})

target_link_libraries(pc_error PRIVATE ${PCL_LIBRARIES})

target_link_libraries(pc_error PRIVATE ${Boost_LIBRARIES})

find_library(PCL_IO_PLY_LIB pcl_io_ply HINTS ${PCL_LIBRARY_DIRS} ${PCL_LIBRARIES})
find_library(PCL_IO_PCD_LIB pcl_io_pcd HINTS ${PCL_LIBRARY_DIRS} ${PCL_LIBRARIES})
if (PCL_IO_PLY_LIB)
  target_link_libraries(pc_error PRIVATE ${PCL_IO_PLY_LIB})
endif()
if (PCL_IO_PCD_LIB)
  target_link_libraries(pc_error PRIVATE ${PCL_IO_PCD_LIB})
endif()

install(TARGETS pc_error DESTINATION ".")
set_target_properties(pc_error PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_OUTPUT_FOLDER}
  RUNTIME_OUTPUT_DIRECTORY_DEBUG   ${PROJECT_OUTPUT_FOLDER}
  DEBUG_POSTFIX "_d"
)

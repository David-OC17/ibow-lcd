cmake_minimum_required(VERSION 3.5)
project(ibow-lcd LANGUAGES CXX)

# Default to Release
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

# Compiler flags
if(CMAKE_BUILD_TYPE MATCHES Release)
  message(STATUS "Setting Release options")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3 -march=native")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -march=native")
elseif(CMAKE_BUILD_TYPE MATCHES Debug)
  message(STATUS "Setting Debug options")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O1 -pg -march=native")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O1 -pg -march=native")
endif()

message(STATUS "Compiler flags: ${CMAKE_CXX_FLAGS}")

# Dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(obindex2 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem)
find_package(OpenMP REQUIRED)

if(OpenMP_CXX_FOUND)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# Includes
include_directories(
  include
  external
  ${OpenCV_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)

# Copy JSON files
file(GLOB JSON "evaluation/*.json")
file(COPY ${JSON} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

# Libraries
add_library(lcdetector
  include/ibow-lcd/island.h
  src/lcdetector.cc
)

ament_target_dependencies(lcdetector
  rclcpp
  obindex2
  OpenCV
  Boost
)

# Executables
add_executable(demo src/main.cc)
ament_target_dependencies(demo
  rclcpp
  obindex2
  OpenCV
  Boost
)
target_link_libraries(demo lcdetector)

add_executable(evaluator
  evaluation/lcevaluator.cc
  evaluation/main.cc
)
ament_target_dependencies(evaluator
  rclcpp
  obindex2
  OpenCV
  Boost
)
target_link_libraries(evaluator lcdetector)

# Installation
install(TARGETS lcdetector demo evaluator
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/ibow-lcd DESTINATION include)

ament_package()

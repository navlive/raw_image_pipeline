#
# Copyright (c) 2021-2023, ETH Zurich, Robotic Systems Lab, Matias Mattamala. All rights reserved.
# Licensed under the MIT license. See LICENSE file in the project root for details.
# 

cmake_minimum_required(VERSION 3.8)
project(raw_image_pipeline_white_balance)

set(CMAKE_CXX_STANDARD 17)
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()


find_package(ament_cmake REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(OpenCV REQUIRED)
find_package(rclcpp REQUIRED)
# ROS2HACK: find dynamic_reconfigure

# Check if OpenCV was compiled with CUDA
if(${OpenCV_CUDA_VERSION})
  add_definitions (-DHAS_CUDA)
endif()

# ROS2HACK: disable dynamic reconfigure
# generate_dynamic_reconfigure_options(
#   cfg/RawImagePipelineWhiteBalance.cfg
# )

include_directories(
  include
  ${OpenCV_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME} SHARED
    src/raw_image_pipeline_white_balance/convolutional_color_constancy.cpp
)
ament_target_dependencies(${PROJECT_NAME} OpenCV)


# Add executable
add_executable(${PROJECT_NAME}_node
    src/raw_image_pipeline_white_balance_node.cpp
)
ament_target_dependencies(${PROJECT_NAME}_node ament_index_cpp rclcpp)
target_link_libraries(${PROJECT_NAME}_node
    ${PROJECT_NAME}
)

# ROS2HACK
# add_dependencies(${PROJECT_NAME}_node ${PROJECT_NAME}_gencfg)

# Mark executables and/or libraries for installation
install(
  TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Mark cpp header files for installation
install(
  DIRECTORY include/
  DESTINATION include
  FILES_MATCHING PATTERN "*.hpp"
                 PATTERN "*.h"
)

# Mark other files for installation
install(
  DIRECTORY model
  DESTINATION share/${PROJECT_NAME}
)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_export_dependencies(Eigen3 OpenCV)

ament_package()

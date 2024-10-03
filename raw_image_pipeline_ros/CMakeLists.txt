#
# Copyright (c) 2021-2023, ETH Zurich, Robotic Systems Lab, Matias Mattamala. All rights reserved.
# Licensed under the MIT license. See LICENSE file in the project root for details.
#

cmake_minimum_required(VERSION 3.8)
project(raw_image_pipeline_ros)

set(CMAKE_CXX_STANDARD 17)
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(cv_bridge REQUIRED)
# find_package(glog_catkin REQUIRED)
find_package(image_transport REQUIRED)
#find_package(nodelet REQUIRED)
find_package(raw_image_pipeline REQUIRED)
find_package(yaml-cpp REQUIRED)

find_package(Eigen3 REQUIRED)
message("Eigen Version:" ${EIGEN3_VERSION_STRING})
message("Eigen Path:" ${Eigen3_DIR})

##################################################################################
# Check if the custom OpenCV directory exists
set(USER_HOME_DIRECTORY $ENV{HOME})
set(OPENCV_CUSTOM_PATH "/usr/local/lib/cmake/opencv4/")
message(STATUS "Searching for custom opencv version in: ${OPENCV_CUSTOM_PATH}")
if(EXISTS "${OPENCV_CUSTOM_PATH}")
  message(STATUS "Custom OpenCV path found at: ${OPENCV_CUSTOM_PATH}")
  set(OpenCV_DIR "${OPENCV_CUSTOM_PATH}")
else()
  message(WARNING "Custom OpenCV path not found. Falling back to default OpenCV.")
endif()

find_package(OpenCV REQUIRED)
message("OpenCV version: ${OpenCV_Version}")
message("OpenCV path: ${OpenCV_DIR}")

include_directories(
  include
  SYSTEM ${EIGEN3_INCLUDE_DIR}
  SYSTEM ${OpenCV_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME} SHARED
    src/raw_image_pipeline_ros.cpp
)
ament_target_dependencies(${PROJECT_NAME}
  cv_bridge
  image_transport
  OpenCV
  raw_image_pipeline
)
target_link_libraries(${PROJECT_NAME} yaml-cpp ${OpenCV_LIBS})

# ROS2HACK
# add_library(${PROJECT_NAME}_nodelet
#     src/raw_image_pipeline_ros_nodelet.cpp
# )
# target_link_libraries(${PROJECT_NAME}_nodelet
#     ${PROJECT_NAME}
# )

add_executable(${PROJECT_NAME}_node
    src/raw_image_pipeline_ros_node.cpp
)
target_link_libraries(${PROJECT_NAME}_node
    ${PROJECT_NAME}
)


install(TARGETS ${PROJECT_NAME}
  DESTINATION lib
)

install(TARGETS ${PROJECT_NAME}_node # ${PROJECT_NAME}_nodelet 
  DESTINATION lib/${PROJECT_NAME}
)

## Mark cpp header files for installation
install(DIRECTORY include/
  DESTINATION include
  FILES_MATCHING PATTERN "*.hpp"
)

install(DIRECTORY config launch 
        DESTINATION share/${PROJECT_NAME})

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME} ${PROJECT_NAME}_nodelet)
ament_export_dependencies(Eigen3 OpenCV)

ament_package()


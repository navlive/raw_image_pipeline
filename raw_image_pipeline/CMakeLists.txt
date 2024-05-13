#
# Copyright (c) 2021-2023, ETH Zurich, Robotic Systems Lab, Matias Mattamala. All rights reserved.
# Licensed under the MIT license. See LICENSE file in the project root for details.
#

cmake_minimum_required(VERSION 3.8)
project(raw_image_pipeline)

set(CMAKE_CXX_STANDARD 17)
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(raw_image_pipeline_white_balance REQUIRED)
# find_package(pybind11_catkin REQUIRED)
find_package(yaml-cpp REQUIRED)

find_package(Eigen3 REQUIRED)
message("Eigen Version:" ${EIGEN3_VERSION_STRING})
message("Eigen Path:" ${Eigen3_DIR})

find_package(OpenCV REQUIRED)
message("OpenCV version: ${OpenCV_Version}")
message("OpenCV path: ${OpenCV_DIR}")

# Check if OpenCV was compiled with CUDA
if(${OpenCV_CUDA_VERSION})
  add_definitions (-DHAS_CUDA)
  find_package(CUDA)
  set(CUDA_npp_LIBRARIES
    ${CUDA_nppc_LIBRARY}
    ${CUDA_nppial_LIBRARY}
    ${CUDA_nppicc_LIBRARY}
    ${CUDA_nppidei_LIBRARY}
    ${CUDA_nppif_LIBRARY}
    ${CUDA_nppig_LIBRARY}
    ${CUDA_nppim_LIBRARY}
    ${CUDA_nppist_LIBRARY}
    ${CUDA_nppisu_LIBRARY}
    ${CUDA_nppitc_LIBRARY}
    ${CUDA_npps_LIBRARY}
  )
else()
  set(CUDA_npp_LIBRARIES "")
  set(CUDA_INCLUDE_DIRS "")
endif()

include_directories(
  include
  SYSTEM ${CUDA_INCLUDE_DIRS}
  SYSTEM ${EIGEN3_INCLUDE_DIR}
  SYSTEM ${OpenCV_INCLUDE_DIRS}
)

# catkin_package(
#   INCLUDE_DIRS include
#   LIBRARIES ${PROJECT_NAME}
#   CATKIN_DEPENDS
#     ${PACKAGE_DEPENDENCIES}
#   DEPENDS
#     OpenCV
#     YAML_CPP
# )
# 
# include_directories(
#   include
#   ${catkin_INCLUDE_DIRS}
#   ${OpenCV_INCLUDE_DIRS}
# )

add_library(${PROJECT_NAME} SHARED
  src/raw_image_pipeline/modules/color_calibration.cpp
  src/raw_image_pipeline/modules/color_enhancer.cpp
  src/raw_image_pipeline/modules/debayer.cpp
  src/raw_image_pipeline/modules/flip.cpp
  src/raw_image_pipeline/modules/gamma_correction.cpp
  src/raw_image_pipeline/modules/undistortion.cpp
  src/raw_image_pipeline/modules/vignetting_correction.cpp
  src/raw_image_pipeline/modules/white_balance.cpp
  src/raw_image_pipeline/raw_image_pipeline.cpp
)
ament_target_dependencies(${PROJECT_NAME}
  Eigen3
  OpenCV
  raw_image_pipeline_white_balance
  yaml-cpp
)
target_link_libraries(${PROJECT_NAME}
  ${CUDA_npp_LIBRARIES}
)

# Mark executables and/or libraries for installation
install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

## Mark cpp header files for installation
install(DIRECTORY include/
  DESTINATION include
  FILES_MATCHING PATTERN "*.hpp"
)

install(DIRECTORY config 
        DESTINATION share/${PROJECT_NAME})

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_export_dependencies(Eigen3 OpenCV raw_image_pipeline_white_balance yaml-cpp)

ament_package()

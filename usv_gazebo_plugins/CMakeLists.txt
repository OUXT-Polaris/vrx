cmake_minimum_required(VERSION 3.5)
project(usv_gazebo_plugins)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# For this package set as C++14
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX14)
    set(CMAKE_CXX_FLAGS "-std=c++14")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "-std=c++0x")
else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++14 support. Please use a different C++ compiler.")
endif()

find_package(ament_cmake REQUIRED)
find_package(gazebo_dev REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(xacro REQUIRED)
find_package(rclcpp REQUIRED)
find_package(wave_gazebo_plugins REQUIRED)

find_package(Eigen3 REQUIRED)

# Plugins require c++11
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")

include_directories(
  include
  ${EIGEN3_INCLUDE_DIRS}
  ${GAZEBO_INCLUDE_DIRS}
  ${rclcpp_INCLUDE_DIRS}
  ${std_msgs_INCLUDE_DIRS}
  ${wave_gazebo_plugins_INCLUDE_DIRS}
)

## Declare a C++ library
add_library(buoyancy_gazebo_plugin
  src/buoyancy_gazebo_plugin.cc
  src/shape_volume.cc
  src/polyhedron_volume.cc
)
target_link_libraries(buoyancy_gazebo_plugin
  ${GAZEBO_LIBRARIES}
  ${Eigen_LIBRARIES}
)

## Declare a C++ library
add_library(usv_gazebo_dynamics_plugin
  src/usv_gazebo_dynamics_plugin.cc
)
target_link_libraries(usv_gazebo_dynamics_plugin
  ${Eigen_LIBRARIES}
)

## Declare a C++ library
add_library(usv_gazebo_thrust_plugin
  src/usv_gazebo_thrust_plugin.cc
)
target_link_libraries(usv_gazebo_thrust_plugin
  ${Eigen_LIBRARIES}
)
ament_target_dependencies(usv_gazebo_thrust_plugin rclcpp std_msgs sensor_msgs)

## Declare a C++ library
add_library(usv_gazebo_wind_plugin
  src/usv_gazebo_wind_plugin.cc
)
target_link_libraries(usv_gazebo_wind_plugin
  ${Eigen_LIBRARIES}
)

# Generate demo world files from xacro and install
xacro_add_files(
  worlds/buoyancy_plugin_demo.world.xacro
  INSTALL DESTINATION worlds
)
install(DIRECTORY worlds/
        DESTINATION worlds)

install(
  DIRECTORY "include/"
  DESTINATION include
)

install(TARGETS
  usv_gazebo_wind_plugin
  usv_gazebo_thrust_plugin
  usv_gazebo_dynamics_plugin
  buoyancy_gazebo_plugin
  DESTINATION lib/${PROJECT_NAME})

ament_package()
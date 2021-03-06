cmake_minimum_required(VERSION 3.5)
project(vrx_gazebo)

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

# We need erb to process the .world.erb files.
find_program(ERB_EXE_PATH erb)
if(NOT ERB_EXE_PATH)
  message(FATAL_ERROR "Could not find the `erb` tool.  Try `sudo apt-get install ruby`")
endif()

set (CMAKE_AUTOMOC ON)
set (CMAKE_AUTOUIC ON)

find_package (Qt5Widgets REQUIRED)
find_package (Qt5Core REQUIRED)
find_package (Protobuf REQUIRED)
find_package (gazebo REQUIRED)


find_package(ament_cmake REQUIRED)
find_package(wamv_gazebo REQUIRED)
find_package(wave_gazebo REQUIRED)
find_package(xacro REQUIRED)
find_package(gazebo_dev REQUIRED)
find_package(geographic_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rclcpp REQUIRED)
find_package(vrx_msgs REQUIRED)

set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS} ${GAZEBO_CXX_FLAGS}") 

include_directories(include
  ${Qt5Core_INCLUDE_DIRS}
  ${QT_USE_FILE}
  ${GAZEBO_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}/msgs
  ${rclcpp_INCLUDE_DIRS}
  ${std_msgs_INCLUDE_DIRS}
  ${vrx_msgs_INCLUDE_DIRS}
)

link_directories(${GAZEBO_LIBRARY_DIRS} ${CMAKE_CURRENT_BINARY_DIR}/msgs)
add_subdirectory(msgs)

add_definitions(${QT_DEFINITIONS})

# Library for displaying waypoint markers
add_library(waypoint_markers src/waypoint_markers.cc)

# Plugin for setting color of light buoy.
add_library(light_buoy_plugin src/light_buoy_plugin.cc)
target_link_libraries(light_buoy_plugin
  light_buoy_colors_msgs
)
ament_target_dependencies(light_buoy_plugin rclcpp std_msgs)
add_dependencies(light_buoy_plugin light_buoy_colors_msgs)

# Plugin for setting color and shape of a placard.
add_library(placard_plugin src/placard_plugin.cc)
target_link_libraries(placard_plugin dock_placard_msgs)
add_dependencies(placard_plugin dock_placard_msgs)

# A generic scoring plugin.
add_library(scoring_plugin src/scoring_plugin.cc)
ament_target_dependencies(scoring_plugin rclcpp vrx_msgs)

# Plugin for scoring the navigation challenge task.
#add_library(navigation_scoring_plugin src/navigation_scoring_plugin.cc)
#target_link_libraries(navigation_scoring_plugin
#  ${catkin_LIBRARIES}
#  scoring_plugin)
#add_dependencies(navigation_scoring_plugin ${catkin_EXPORTED_TARGETS})
#install(TARGETS navigation_scoring_plugin
#  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

# Plugin for scoring the station keeping challenge task.
#add_library(stationkeeping_scoring_plugin src/stationkeeping_scoring_plugin.cc)
#target_link_libraries(stationkeeping_scoring_plugin
#  ${catkin_LIBRARIES}
#  scoring_plugin
#  waypoint_markers)
#add_dependencies(stationkeeping_scoring_plugin ${catkin_EXPORTED_TARGETS})
#install(TARGETS stationkeeping_scoring_plugin
#  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

# Plugin for scoring the waypoint navigation challenge task.
#add_library(wayfinding_scoring_plugin src/wayfinding_scoring_plugin.cc)
#target_link_libraries(wayfinding_scoring_plugin
#  ${catkin_LIBRARIES}
#  scoring_plugin
#  waypoint_markers)
#add_dependencies(wayfinding_scoring_plugin ${catkin_EXPORTED_TARGETS})
#install(TARGETS wayfinding_scoring_plugin
#  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

# Plugin for scoring the scan and dock task.
#add_library(scan_dock_scoring_plugin src/scan_dock_scoring_plugin.cc)
#target_link_libraries(scan_dock_scoring_plugin
#  ${catkin_LIBRARIES}
#  scoring_plugin 
#  light_buoy_colors_msgs
#  dock_placard_msgs
#  )
#add_dependencies(scan_dock_scoring_plugin
#  ${catkin_EXPORTED_TARGETS}
#  light_buoy_colors_msgs
#  dock_placard_msgs
#)
#install(TARGETS scan_dock_scoring_plugin
#  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

#if("${GAZEBO_MAJOR_VERSION}" GREATER "7")
#  # Plugin for Task Info GUI Overlay
#  add_library(gui_task_widget SHARED ${headers_MOC}
#         	src/gui_task_widget.cc
#  )

#  target_link_libraries(gui_task_widget 
#    ${catkin_LIBRARIES}
#    ${GAZEBO_LIBRARIES}
#    ${PROTOBUF_LIBRARIES}
#    ${QT_LIBRARIES}
#  )
#  qt5_use_modules(gui_task_widget Widgets)
#  add_dependencies(gui_task_widget ${catkin_EXPORTED_TARGETS})
#  install(TARGETS gui_task_widget
#    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#  )
#endif()

# Plugin for scoring the perception task.
#add_library(perception_scoring_plugin src/perception_scoring_plugin.cc)
#target_link_libraries(perception_scoring_plugin
#  ${catkin_LIBRARIES}
#  scoring_plugin)
#add_dependencies(perception_scoring_plugin ${catkin_EXPORTED_TARGETS})
#install(TARGETS perception_scoring_plugin
#  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

install(TARGETS
  waypoint_markers
  light_buoy_plugin
  placard_plugin
  scoring_plugin
  DESTINATION lib/${PROJECT_NAME})

# Dock base files that need to be processed with erb
set (dock_base_erb_files
  models/dock_2016_base/model.sdf.erb
  models/dock_2018_base/model.sdf.erb
  models/dock_2016_base_dynamic/model.sdf.erb
  models/dock_2018_base_dynamic/model.sdf.erb
)

# Dock files that need to be processed with erb
set (dock_erb_files
  models/dock_2016/model.sdf.erb
  models/dock_2018/model.sdf.erb
  models/dock_2016_dynamic/model.sdf.erb
  models/dock_2018_dynamic/model.sdf.erb
)

# Process the dock base erb files
foreach(_erb ${dock_base_erb_files})
  string(REGEX REPLACE ".sdf.erb" ".sdf" _model ${_erb})
  set(_model ${CMAKE_CURRENT_SOURCE_DIR}/${_model})
  add_custom_command(OUTPUT ${_model}
                     COMMAND ${ERB_EXE_PATH} ${_erb} > ${_model}
                     DEPENDS
                       ${CMAKE_CURRENT_SOURCE_DIR}/${_erb}
                       ${CMAKE_CURRENT_SOURCE_DIR}/dock_generator.erb
                     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

  list(APPEND dock_base_files ${_model})
endforeach()
add_custom_target(dock_base_erb_generation ALL DEPENDS ${dock_base_files})

# Process the dock erb files
foreach(_erb ${dock_erb_files})
  string(REGEX REPLACE ".sdf.erb" ".sdf" _model ${_erb})
  set(_model ${CMAKE_CURRENT_SOURCE_DIR}/${_model})
  add_custom_command(OUTPUT ${_model}
                     COMMAND ${ERB_EXE_PATH} ${_erb} > ${_model}
                     DEPENDS
                       ${CMAKE_CURRENT_SOURCE_DIR}/${_erb}
                       ${CMAKE_CURRENT_SOURCE_DIR}/dock_generator.erb
                     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

  list(APPEND dock_files ${_model})
endforeach()
add_custom_target(dock_erb_generation ALL
  DEPENDS
    ${dock_files}
    dock_base_erb_generation
)

# Install all the world files
install(DIRECTORY worlds/
  DESTINATION share/vrx_gazebo/worlds)

# Install config files
install(DIRECTORY config/
  DESTINATION share/vrx_gazebo/config)

# Install all the model files
install(DIRECTORY models/
  DESTINATION share/vrx_gazebo/models)

# Install all the launch files
install(DIRECTORY launch/
  DESTINATION share/vrx_gazebo)

# Generate world files from xacro and install
xacro_add_files(
  worlds/example_course.world.xacro
  worlds/navigation_task.world.xacro
  worlds/perception_task.world.xacro
  worlds/sandisland.world.xacro
  worlds/dock.world.xacro
  worlds/scan_and_dock.world.xacro
  worlds/stationkeeping_task.world.xacro
  worlds/wayfinding_task.world.xacro
  worlds/wind_test.world.xacro
  worlds/ocean.world.xacro
  INSTALL DESTINATION worlds
)

# Generate obstacle course
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/models/robotx_2018_qualifying_avoid_obstacles_buoys/model.sdf
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/scripts/generate_avoid_obstacles_buoys
  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/scripts/generate_avoid_obstacles_buoys --seed 1337 --a3 6 --a5 7 --a7 7 > ${CMAKE_CURRENT_SOURCE_DIR}/models/robotx_2018_qualifying_avoid_obstacles_buoys/model.sdf
)
add_custom_target(${PROJECT_NAME}_generate_obstacle_course_buoys ALL DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/models/robotx_2018_qualifying_avoid_obstacles_buoys/model.sdf)

#install(PROGRAMS scripts/spawn_wamv.bash
#  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

ament_package()
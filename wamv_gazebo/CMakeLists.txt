cmake_minimum_required(VERSION 3.5)
project(wamv_gazebo)

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

find_package(ament_cmake REQUIRED)
find_package(wamv_description REQUIRED)
find_package(usv_gazebo_plugins REQUIRED)
find_package(xacro REQUIRED)

# Generate urdf files from xacro and install
xacro_add_files(
  urdf/wamv_gazebo.urdf.xacro
  INSTALL DESTINATION urdf
)

# Install meshes and textures
install(DIRECTORY models/
  DESTINATION share/wamv_gazebo/models)

# Install xacro files / macros
install(DIRECTORY urdf/
  DESTINATION share/wamv_gazebo/urdf)

# Install config files
install(DIRECTORY config/
  DESTINATION share/wamv_gazebo/config)

# Install launch files
install(DIRECTORY launch/
  DESTINATION share/wamv_gazebo/launch)

ament_package()
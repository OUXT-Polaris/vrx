cmake_minimum_required(VERSION 2.8.3)
project(wamv_description)
find_package(ament_cmake REQUIRED)
find_package(xacro REQUIRED)

xacro_add_files(
  urdf/wamv_base.urdf.xacro
    INSTALL DESTINATION urdf
)

install(DIRECTORY models/
  DESTINATION share/models)

install(DIRECTORY urdf/
  DESTINATION share/urdf)
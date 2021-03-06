# Copyright (C) 2017-2020 LAAS-CNRS
#
# Author: Olivier Stasse
# Author: Guilhem Saurel
#
CMAKE_MINIMUM_REQUIRED(VERSION 3.1)

# Project properties
SET(PROJECT_ORG stack-of-tasks)
SET(PROJECT_DESCRIPTION roscontrol_sot)
SET(PROJECT_NAME roscontrol_sot)
SET(PROJECT_URL "https://github.com/${PROJECT_ORG}/${PROJECT_NAME}")

# Project options

# Project configuration
SET(CXX_DISABLE_WERROR True)
SET(PROJECT_USE_CMAKE_EXPORT TRUE)

# JRL-cmakemodule setup
INCLUDE(cmake/base.cmake)
INCLUDE(cmake/boost.cmake)
INCLUDE(cmake/python.cmake)
INCLUDE(cmake/ros.cmake)

# Project definition
COMPUTE_PROJECT_ARGS(PROJECT_ARGS LANGUAGES CXX)
PROJECT(${PROJECT_NAME} ${PROJECT_ARGS})
CHECK_MINIMAL_CXX_STANDARD(11 ENFORCE)

# Project dependencies
FINDPYTHON()
SEARCH_FOR_BOOST_PYTHON()
add_required_dependency(bullet)
add_required_dependency(urdfdom)

SET(CATKIN_REQUIRED_COMPONENTS
  temperature_sensor_controller
  pal_hardware_interfaces
  controller_interface
  controller_manager
  pal_common_msgs
  roscpp
  std_msgs
  control_msgs
  sensor_msgs
  realtime_tools
  rospy
  )

find_package(catkin REQUIRED COMPONENTS ${CATKIN_REQUIRED_COMPONENTS})

ADD_PROJECT_DEPENDENCY(dynamic-graph-python 4.0.0 REQUIRED)
ADD_PROJECT_DEPENDENCY(dynamic_graph_bridge REQUIRED)
ADD_PROJECT_DEPENDENCY(sot-core REQUIRED )
ADD_PROJECT_DEPENDENCY(pinocchio REQUIRED)
ADD_PROJECT_DEPENDENCY(control_toolbox REQUIRED)

# Detect the controller interface version to switch code
if(controller_interface_FOUND)
  if (${controller_interface_VERSION} VERSION_GREATER "0.2.5")
    add_definitions(-DCONTROLLER_INTERFACE_KINETIC)
  endif(${controller_interface_VERSION} VERSION_GREATER "0.2.5")
endif(controller_interface_FOUND)

# Detect if temperature sensor controller package is found
# if yes then it is a PAL Robotics Forked code.
if(temperature_sensor_controller_FOUND)
  add_definitions(-DTEMPERATURE_SENSOR_CONTROLLER)
endif(temperature_sensor_controller_FOUND)

add_library(rcsot_controller
  src/roscontrol-sot-controller.cpp
  src/log.cpp
  )
TARGET_INCLUDE_DIRECTORIES(rcsot_controller SYSTEM PUBLIC ${catkin_INCLUDE_DIRS})
TARGET_INCLUDE_DIRECTORIES(rcsot_controller PUBLIC $<INSTALL_INTERFACE:include>)
TARGET_LINK_LIBRARIES(rcsot_controller ${catkin_LIBRARIES} ${bullet_LIBRARIES}
  sot-core::sot-core dynamic_graph_bridge::sot_loader
  ${control_toolbox_LIBRARIES})

IF(SUFFIX_SO_VERSION)
  SET_TARGET_PROPERTIES(rcsot_controller PROPERTIES SOVERSION ${PROJECT_VERSION})
ENDIF(SUFFIX_SO_VERSION)

IF(NOT INSTALL_PYTHON_INTERFACE_ONLY)
  INSTALL(TARGETS rcsot_controller EXPORT ${TARGETS_EXPORT_NAME} DESTINATION lib)
ENDIF(NOT INSTALL_PYTHON_INTERFACE_ONLY)

ADD_EXECUTABLE(roscontrol-sot-parse-log src/roscontrol-sot-parse-log.cc)
install(TARGETS roscontrol-sot-parse-log EXPORT ${TARGETS_EXPORT_NAME} DESTINATION bin)

foreach(dir config launch)
  install(DIRECTORY ${dir} DESTINATION share/${PROJECT_NAME})
endforeach(dir)

ADD_SUBDIRECTORY(tests)
INSTALL(FILES package.xml DESTINATION share/${PROJECT_NAME})

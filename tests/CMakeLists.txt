# Copyright (C) 2017-2019 LAAS-CNRS
#
# Author: Olivier Stasse
#

ADD_EXECUTABLE(roscontrol_sot_hardware roscontrol_sot_hardware.cpp)
TARGET_INCLUDE_DIRECTORIES(roscontrol_sot_hardware PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)
TARGET_LINK_LIBRARIES(roscontrol_sot_hardware rcsot_controller
  ${catkin_LIBRARIES} ${bullet_libraries})

# Add the library to wrap the test device.
SET(DEVICE_NAME sot-test-device)
ADD_LIBRARY(${DEVICE_NAME} SHARED ${DEVICE_NAME}.cpp)
TARGET_LINK_LIBRARIES(${DEVICE_NAME} sot-core::sot-core)
INSTALL(TARGETS ${DEVICE_NAME} DESTINATION lib)

# build python submodule
STRING(REPLACE - _ PYTHON_LIBRARY_NAME ${DEVICE_NAME})

DYNAMIC_GRAPH_PYTHON_MODULE("sot/test/${PYTHON_LIBRARY_NAME}"
  ${DEVICE_NAME} sot-test-${PYTHON_LIBRARY_NAME}-wrap
  MODULE_HEADER "${CMAKE_CURRENT_SOURCE_DIR}/${DEVICE_NAME}-python.hh")
SET_TARGET_PROPERTIES(sot-test-${PYTHON_LIBRARY_NAME}-wrap PROPERTIES
  LIBRARY_OUTPUT_DIRECTORY ../src) # FIXME: weird catkin / dg interraction

# Add the library to wrap the controller of Test.
SET(CONTROLLER_NAME sot-test-controller)
ADD_LIBRARY(${CONTROLLER_NAME} SHARED sot-test-controller.cpp)
TARGET_INCLUDE_DIRECTORIES(${CONTROLLER_NAME} SYSTEM PUBLIC
  ${catkin_INCLUDE_DIRS} ${PYTHON_INCLUDE_DIRS})
TARGET_LINK_LIBRARIES(${CONTROLLER_NAME} ${DEVICE_NAME})
INSTALL(TARGETS ${CONTROLLER_NAME} DESTINATION lib)

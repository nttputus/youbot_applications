PROJECT(youBot_JointConfigurator)
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)


SET(ROS_ROOT_PATH $ENV{ROS_ROOT})

IF(DEFINED ROS_ROOT_PATH)
	include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)
	rosbuild_find_ros_package(soem)
  rosbuild_find_ros_package(youbot_driver)
ENDIF(DEFINED ROS_ROOT_PATH)

## check required prerequisites
SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/external/cmake_modules")
SET(Boost_USE_STATIC_LIBS ON)
SET(Boost_ADDITIONAL_VERSIONS "1.39" "1.39.0" "1.40" "1.40.0" "1.41" "1.41.0" "1.42" "1.42.0" "1.43" "1.43.0" "1.44" "1.44.0" "1.45" "1.45.0" "1.46" "1.46.0")
FIND_PACKAGE(Boost REQUIRED COMPONENTS thread date_time filesystem system)
FIND_PACKAGE(Soem REQUIRED)
FIND_PACKAGE(OODL_YouBot REQUIRED)


## preprocessor definitions
SET(CMAKE_C_FLAGS_DEBUG "-fPIC -Wall -g")
SET(CMAKE_CXX_FLAGS_DEBUG "-fPIC -O0 -g")

IF(NOT DEFINED JOINT_CONFIGURATOR_CONFIG_DIR)
  SET(JOINT_CONFIGURATOR_CONFIG_DIR ${CMAKE_HOME_DIRECTORY}/config/)
ENDIF(NOT DEFINED JOINT_CONFIGURATOR_CONFIG_DIR)

ADD_DEFINITIONS(-DYOUBOT_CONFIGURATIONS_DIR="${OODL_YOUBOT_CONFIG_DIR}")
ADD_DEFINITIONS(-DCONFIG_DIR="${JOINT_CONFIGURATOR_CONFIG_DIR}")

## build parameters and paths
SET(CMAKE_BUILD_TYPE DEBUG) #enable debug mode (e.g. for embedded gdb in eclipse )
SET(EXECUTABLE_OUTPUT_PATH ${CMAKE_HOME_DIRECTORY}/bin CACHE PATH "Configure the executable output path.")
SET(LIBRARY_OUTPUT_PATH ${CMAKE_HOME_DIRECTORY}/lib CACHE PATH "Configure the library output path.")

## Set include directories
INCLUDE_DIRECTORIES(
	${OODL_YOUBOT_INCLUDE_DIR}
	${SOEM_INCLUDE_DIR}
	${Boost_INCLUDE_DIR}
)

## Compile and link
ADD_EXECUTABLE(youBot_JointConfigurator
  src/JointConfiguratorMain.cpp
)

ADD_EXECUTABLE(youBot_ArmConfigurator
  src/ArmConfigurator.cpp
)

ADD_EXECUTABLE(youBot_BaseConfigurator
  src/BaseConfigurator.cpp
)

ADD_LIBRARY(JointConfigurator
  src/JointConfigurator.cpp
)

ADD_LIBRARY(JointConfiguratorShared SHARED)
SET_TARGET_PROPERTIES(JointConfiguratorShared PROPERTIES OUTPUT_NAME "JointConfigurator")
TARGET_LINK_LIBRARIES(JointConfiguratorShared -Wl,-whole-archive JointConfigurator -Wl,-no-whole-archive)

SET(YOUBOT_DRIVER_CONFIG
${CMAKE_HOME_DIRECTORY}/config/arm.cfg
${CMAKE_HOME_DIRECTORY}/config/arm-1-protected.cfg
${CMAKE_HOME_DIRECTORY}/config/arm-2-protected.cfg
${CMAKE_HOME_DIRECTORY}/config/arm-3-protected.cfg
${CMAKE_HOME_DIRECTORY}/config/arm-4-protected.cfg
${CMAKE_HOME_DIRECTORY}/config/arm-5-protected.cfg
${CMAKE_HOME_DIRECTORY}/config/base.cfg
${CMAKE_HOME_DIRECTORY}/config/base-protected.cfg
)


TARGET_LINK_LIBRARIES(youBot_JointConfigurator
	${OODL_YOUBOT_LIBRARIES}
	${SOEM_LIBRARIES}
	${Boost_LIBRARIES}
	pthread
  JointConfigurator
)

TARGET_LINK_LIBRARIES(youBot_ArmConfigurator
	${OODL_YOUBOT_LIBRARIES}
	${SOEM_LIBRARIES}
	${Boost_LIBRARIES}
	pthread
  JointConfigurator
)

TARGET_LINK_LIBRARIES(youBot_BaseConfigurator
	${OODL_YOUBOT_LIBRARIES}
	${SOEM_LIBRARIES}
	${Boost_LIBRARIES}
	pthread
  JointConfigurator
)

MACRO(INSTALL_CONFIG_WITH_DIRECTORY CONFIG_LIST)
  FOREACH(CONFIG ${${CONFIG_LIST}})
    STRING(REGEX MATCH "(.*)[/\\]" DIR ${CONFIG})
    INSTALL(FILES ${CONFIG} DESTINATION config)
  ENDFOREACH(CONFIG)
ENDMACRO(INSTALL_CONFIG_WITH_DIRECTORY)


INSTALL_CONFIG_WITH_DIRECTORY(YOUBOT_DRIVER_CONFIG)

INSTALL(TARGETS youBot_JointConfigurator youBot_ArmConfigurator RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)

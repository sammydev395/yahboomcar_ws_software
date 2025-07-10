cmake_minimum_required(VERSION 3.0.2)
project(orb_slam2)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  sensor_msgs
  cv_bridge
  image_transport
  tf
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS roscpp sensor_msgs cv_bridge image_transport tf
)

IF(NOT ROS_BUILD_TYPE)
  SET(ROS_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${ROS_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall  -O3 -march=native")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-reorder")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../../../cmake_modules)

find_package(OpenCV 4.2.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_SOURCE_DIR}/../../../
  ${PROJECT_SOURCE_DIR}/../../../include
  ${Pangolin_INCLUDE_DIRS}
)

set(LIBS 
${OpenCV_LIBS} 
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/../../../Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/../../../Thirdparty/g2o/lib/libg2o.so
${PROJECT_SOURCE_DIR}/../../../lib/libORB_SLAM2.so
-lboost_system
)

# Node for monocular camera
add_executable(Mono
src/ros_mono.cc
)

target_link_libraries(Mono
${LIBS}
${catkin_LIBRARIES}
)

# Node for monocular camera (Augmented Reality Demo)
add_executable(MonoAR
src/AR/ros_mono_ar.cc
src/AR/ViewerAR.h
src/AR/ViewerAR.cc
)

target_link_libraries(MonoAR
${LIBS}
${catkin_LIBRARIES}
)

# Node for stereo camera
add_executable(Stereo
src/ros_stereo.cc
)

target_link_libraries(Stereo
${LIBS}
${catkin_LIBRARIES}
)

# Node for RGB-D camera
add_executable(RGBD
src/ros_rgbd.cc
)

target_link_libraries(RGBD
${LIBS}
${catkin_LIBRARIES}
)

# Node for RGB-D-pose camera
add_executable(RGBD_pose
src/ros_rgbd_pose.cc
)

target_link_libraries(RGBD_pose
${LIBS}
${catkin_LIBRARIES}
)



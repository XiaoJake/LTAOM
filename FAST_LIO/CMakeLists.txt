cmake_minimum_required(VERSION 2.8.3)
project(fast_lio)

SET(CMAKE_BUILD_TYPE "Release")

ADD_COMPILE_OPTIONS(-std=c++14 )
ADD_COMPILE_OPTIONS(-std=c++14 )
set( CMAKE_CXX_FLAGS "-std=c++14 -O3" )

add_definitions(-DROOT_DIR=\"${CMAKE_CURRENT_SOURCE_DIR}/\")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fexceptions" )
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -pthread -std=c++0x -std=c++14 -fexceptions")

message("Current CPU archtecture: ${CMAKE_SYSTEM_PROCESSOR}")
if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)" )
  include(ProcessorCount)
  ProcessorCount(N)
  message("Processer number:  ${N}")
  if(N GREATER 5)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=4)
    message("core for MP:  3")
  elseif(N GREATER 3)
    math(EXPR PROC_NUM "${N} - 2")
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM="${PROC_NUM}")
    message("core for MP:  ${PROC_NUM}")
  else()
    add_definitions(-DMP_PROC_NUM=1)
  endif()
else()
  add_definitions(-DMP_PROC_NUM=1)
endif()

find_package(OpenMP QUIET)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")

find_package(PythonLibs REQUIRED)
find_path(MATPLOTLIB_CPP_INCLUDE_DIRS "matplotlibcpp.h")

find_package(OpenCV REQUIRED)

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  nav_msgs
  sensor_msgs
  roscpp
  rospy
  std_msgs
  pcl_ros
  tf
  livox_ros_driver2
  message_generation
  eigen_conversions
  nodelet
)

find_package(Eigen3 REQUIRED)
find_package(PCL 1.8 REQUIRED)

message(Eigen: ${EIGEN3_INCLUDE_DIR})

include_directories(
	${catkin_INCLUDE_DIRS} 
  ${EIGEN3_INCLUDE_DIR}
  ${PCL_INCLUDE_DIRS}
  ${PYTHON_INCLUDE_DIRS}
  include
  src)

add_message_files(
  FILES
  Pose6D.msg
  States.msg
)

generate_messages(
 DEPENDENCIES
 geometry_msgs
)

catkin_package(
  CATKIN_DEPENDS geometry_msgs nav_msgs roscpp rospy std_msgs message_runtime nodelet
  DEPENDS EIGEN3 PCL
  INCLUDE_DIRS
)

# add_executable(loam_feat_extract src/feature_extract.cpp)
# target_link_libraries(loam_feat_extract ${catkin_LIBRARIES} ${PCL_LIBRARIES})
add_library(extra_lib src/extra_lib.cpp)
target_link_libraries(extra_lib ${OpenCV_LIBS} ${OpenCV_LIBRARIES})

#add_executable(fastlio_mapping src/laserMapping.cpp include/ikd-Tree/ikd_Tree.cpp include/ikd-Forest/ikd_Forest.cpp include/FOV_Checker/FOV_Checker.cpp src/preprocess.cpp)
add_library(fastlio_mapping src/laserMapping.cpp include/ikd-Tree/ikd_Tree.cpp include/ikd-Forest/ikd_Forest.cpp include/FOV_Checker/FOV_Checker.cpp src/preprocess.cpp)
add_dependencies(fastlio_mapping ${fastlio_mapping_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(fastlio_mapping extra_lib ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${PYTHON_LIBRARIES} ${OpenCV_LIBS})
target_include_directories(fastlio_mapping PRIVATE ${PYTHON_INCLUDE_DIRS})

add_library(fastlio_plugin
  src/fastlio_plugin.cpp
)
target_link_libraries(fastlio_plugin fastlio_mapping ${catkin_LIBRARIES})
if(catkin_EXPORTED_LIBRARIES)
  add_dependencies(fastlio_plugin ${fastlio_plugin_EXPORTED_TARGETS} ${catkin_EXPORTED_LIBRARIES})
endif()
# add_executable(kd_tree_test include/ikd-Tree/ikd_Tree.cpp src/kd_tree_test.cpp)
# target_link_libraries(kd_tree_test ${PCL_LIBRARIES})

# add_executable(ikd_Forest_Test include/ikd-Forest/ikd_Forest.cpp test/ikd_Forest_Test.cpp)
# target_link_libraries(ikd_Forest_Test ${PCL_LIBRARIES})

# add_executable(fov_test test/fov_test.cpp include/FOV_Checker/FOV_Checker.cpp)

add_executable(map_eva map_eva/mapconsistency_eva.cpp)
target_link_libraries(map_eva ${catkin_LIBRARIES} ${PCL_LIBRARIES})


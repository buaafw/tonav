cmake_minimum_required(VERSION 2.8)

project(tonav)
set (tonav_CMAKE_DIR "${tonav_SOURCE_DIR}/cmake")
set (CMAKE_MODULE_PATH ${tonav_CMAKE_DIR} ${CMAKE_MODULE_PATH})

find_package(catkin REQUIRED
             COMPONENTS message_generation std_msgs sensor_msgs image_transport)

include_directories("${tonav_SOURCE_DIR}/include")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall")

# set (CMAKE_INSTALL_RPATH /opt/ros/jade/lib)

set(COMMON_SOURCE_FILES
        src/calibration.cpp
        include/calibration.h
        include/filter.h
        include/exceptions/calibration_file_error.h
        src/exceptions/calibration_file_error.cpp
        include/exceptions/impossible_exception.h
        src/exceptions/impossible_exception.cpp
        include/exceptions/base_exception.h
        src/exceptions/base_exception.cpp
        include/imu_item.h
        include/imu_device.h
        src/imu_device.cpp
        include/exceptions/general_exception.h
        src/exceptions/general_exception.cpp
        src/imu_item.cpp
        include/camera_item.h
        src/camera_item.cpp
        include/imu_buffer.h
        src/imu_buffer.cpp
        src/filter.cpp
        include/filter_state.h
        src/filter_state.cpp
        include/feature_tracker.h
        src/feature_tracker.cpp
        include/frame_features.h
        src/frame_features.cpp
        include/camera_pose.h
        include/feature_track.h
        src/feature_track.cpp
        src/camera_pose.cpp
        include/tonav.h
        src/tonav.cpp
        include/body_state.h
        src/body_state.cpp
        include/camera_pose_buffer.h
        src/camera_pose_buffer.cpp
        include/quaternion_tools.h
        src/quaternion_tools.cpp
        include/camera_reprojection_functor.h
        src/camera_reprojection_functor.cpp
        include/feature_rezidualization_result.h
        src/feature_rezidualization_result.cpp
        include/chi_squared_ppm.h
        include/state_initializer.h
        src/state_initializer.cpp
        include/stats.h
        src/stats.cpp
        include/stats_node.h
        src/stats_node.cpp
        include/stats_timer.h
        src/stats_timer.cpp
        include/camera_algorithms.h
        src/camera_algorithms.cpp
        include/quaternion.h
        src/quaternion.cpp
      )
set (TONAV_ROS_SOURCE_FILES
    include/tonav_ros.h
    src/tonav_ros.cpp
    src/main.cpp
  )
add_executable(tonav ${TONAV_ROS_SOURCE_FILES} ${COMMON_SOURCE_FILES})

set (TONAV_KITTI_SOURCE_FILES
    include/tonav_kitti.h
    src/tonav_kitti.cpp
    src/main_kitti.cpp
  )
add_executable(tonavkitti ${TONAV_KITTI_SOURCE_FILES} ${COMMON_SOURCE_FILES})

find_package(Eigen3 REQUIRED)
include_directories (AFTER "${EIGEN3_INCLUDE_DIR}")

catkin_package(
    CATKIN_DEPENDS message_runtime std_msgs sensor_msgs tf2 tf2_ros roscpp
    DEPENDS eigen
)

find_package(Boost REQUIRED COMPONENTS program_options filesystem)
include_directories (AFTER "${Boost_INCLUDE_DIRS}")
target_link_libraries(tonav ${Boost_LIBRARIES})
target_link_libraries(tonavkitti ${Boost_LIBRARIES})

find_package(OpenCV REQUIRED)
message (STATUS "${OpenCV_INCLUDE_DIRS}")
message (STATUS "${OpenCV_LIBS}")
include_directories (AFTER "${OpenCV_INCLUDE_DIRS}")
target_link_libraries(tonav ${OpenCV_LIBS})
target_link_libraries(tonavkitti ${OpenCV_LIBS})

find_package(image_transport REQUIRED)
include_directories (AFTER ${image_transport_INCLUDE_DIRS})
target_link_libraries(tonav ${image_transport_LIBRARIES})

find_package(cv_bridge REQUIRED)
target_link_libraries(tonav ${cv_bridge_LIBRARIES})

find_package(tf2_ros REQUIRED)
target_link_libraries(tonav ${tf2_ros_LIBRARIES})
target_link_libraries(tonavkitti ${tf2_ros_LIBRARIES})

find_package(eigen_conversions REQUIRED)
target_link_libraries(tonav ${eigen_conversions_LIBRARIES})
target_link_libraries(tonavkitti ${eigen_conversions_LIBRARIES})

find_package(Doxygen 1.8.6)
if(DOXYGEN_FOUND)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/deploy.sh.in ${CMAKE_CURRENT_BINARY_DIR}/deploy.sh @ONLY)
    add_custom_target(doc ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generating API documentation with Doxygen" VERBATIM
    )
endif(DOXYGEN_FOUND)

if(CATKIN_ENABLE_TESTING)
  find_package(rostest REQUIRED)
  SET(TEST_FILES
    test/body_state_test.cpp
    test/quaternion_test.cpp
    test/quaternion_tools_test.cpp
    test/filter_test.cpp
    test/big_camera_test.cpp
    test/frame_features_test.cpp
    test/stats_test.cpp
    )
  add_rostest_gtest(${PROJECT_NAME}-test launch/gtest.test ${COMMON_SOURCE_FILES} ${TEST_FILES})
  target_link_libraries(${PROJECT_NAME}-test ${catkin_LIBRARIES} ${GTEST_BOTH_LIBRARIES} ${GMOCK_LIBRARIES} ${Boost_LIBRARIES} ${OpenCV_LIBS} ${image_transport_LIBRARIES} ${cv_bridge_LIBRARIES} ${tf2_ros_LIBRARIES} ${eigen_conversions_LIBRARIES})
endif()

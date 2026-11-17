cmake_minimum_required(VERSION 3.8)
project(sura_bt)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(std_srvs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(diagnostic_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(lifecycle_msgs REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(behaviortree_cpp_v3 REQUIRED)
find_package(controller_manager_msgs REQUIRED)
find_package(sura_actions REQUIRED)
find_package(sura_safety REQUIRED)
find_package(sura_msgs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(yaml-cpp REQUIRED)
find_package(nlohmann_json REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(TINYXML2 REQUIRED IMPORTED_TARGET tinyxml2)

set(SURA_BT_NODE_SOURCES
  src/fleet_tree.cpp
  src/controller_switch_nodes.cpp
  src/operation_mode_nodes.cpp
  src/camera_nodes.cpp
  src/command_nodes.cpp
  src/sura_actions_nodes.cpp
  src/avoid_obstacle.cpp
  src/seafloor_anomaly_nodes.cpp
  src/register_nodes.cpp
)

add_executable(bt_runner
  ${SURA_BT_NODE_SOURCES}
  src/safety_ask_coordinator.cpp
  src/bt_runner.cpp
)

add_executable(export_bt_catalog
  ${SURA_BT_NODE_SOURCES}
  src/export_bt_catalog.cpp
)

foreach(target IN ITEMS bt_runner export_bt_catalog)
  target_include_directories(${target} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
  )
  target_compile_features(${target} PUBLIC cxx_std_17)
  ament_target_dependencies(${target}
    rclcpp
    rclcpp_action
    std_srvs
    std_msgs
    sensor_msgs
    geometry_msgs
    diagnostic_msgs
    cv_bridge
    lifecycle_msgs
    ament_index_cpp
    behaviortree_cpp_v3
    controller_manager_msgs
    sura_actions
    sura_safety
    sura_msgs
    tf2_ros
    tf2_geometry_msgs
  )
  target_link_libraries(${target}
    nlohmann_json::nlohmann_json
    PkgConfig::TINYXML2
  )
endforeach()

target_link_libraries(bt_runner yaml-cpp)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  DIRECTORY trees
  DESTINATION share/${PROJECT_NAME}
)

install(PROGRAMS scripts/safety_ask_bridge.py
  DESTINATION lib/${PROJECT_NAME})

install(
  TARGETS
    bt_runner
    export_bt_catalog
  DESTINATION lib/${PROJECT_NAME}
)

ament_package()

cmake_minimum_required(VERSION 3.5)
project(hardware_interface)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra)
endif()

find_package(ament_cmake REQUIRED)
find_package(control_msgs REQUIRED)
find_package(rclcpp REQUIRED)

add_library(
  hardware_interface
  SHARED
  src/joint_command_handle.cpp
  src/joint_state_handle.cpp
  src/operation_mode_handle.cpp
  src/robot_hardware.cpp
)
target_include_directories(
  hardware_interface
  PUBLIC
  include)
ament_target_dependencies(
  hardware_interface
  control_msgs
  rclcpp
  rcpputils
)

# Causes the visibility macros to use dllexport rather than dllimport,
# which is appropriate when building the dll but not consuming it.
target_compile_definitions(hardware_interface PRIVATE "HARDWARE_INTERFACE_BUILDING_DLL")

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS
  hardware_interface
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()

  find_package(ament_cmake_gmock REQUIRED)

  ament_add_gmock(test_macros test/test_macros.cpp)
  target_include_directories(test_macros PRIVATE include)

  ament_add_gmock(test_robot_hardware_interfaces test/test_robot_hardware_interface.cpp)
  target_include_directories(test_robot_hardware_interfaces PRIVATE include)
  target_link_libraries(test_robot_hardware_interfaces hardware_interface)

  ament_add_gmock(test_register_actuators test/test_register_actuators.cpp)
  target_include_directories(test_register_actuators PRIVATE include)
  target_link_libraries(test_register_actuators hardware_interface)
  ament_target_dependencies(test_register_actuators rcpputils)

  ament_add_gmock(test_actuator_handle test/test_actuator_handle.cpp)
  target_include_directories(test_actuator_handle PRIVATE include)
  target_link_libraries(test_actuator_handle hardware_interface)
  ament_target_dependencies(test_actuator_handle rcpputils)

endif()

ament_export_include_directories(
  include
)
ament_export_libraries(
  hardware_interface)
ament_export_dependencies(
  control_msgs)
ament_package()

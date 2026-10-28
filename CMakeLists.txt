cmake_minimum_required(VERSION 3.10)
project(pmm_planner LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
add_definitions(-DROOT_DIR=\"${CMAKE_CURRENT_SOURCE_DIR}/\")

find_package(ament_cmake REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(laser_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(Threads REQUIRED)
find_package(yaml-cpp REQUIRED)

add_library(pmm_planner SHARED
  src/pmm_mg_trajectory3d.cpp
  src/pmm_trajectory.cpp
  src/pmm_trajectory3d.cpp
  src/common.cpp
  src/pmm_planner.cpp
)
target_include_directories(pmm_planner PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  ${EIGEN3_INCLUDE_DIRS}
)
ament_target_dependencies(pmm_planner PUBLIC Eigen3 laser_msgs nav_msgs)
target_link_libraries(pmm_planner PUBLIC Threads::Threads yaml-cpp)

add_executable(main src/main.cpp)
target_link_libraries(main pmm_planner)

install(TARGETS pmm_planner EXPORT export_pmm_planner
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)
install(TARGETS main RUNTIME DESTINATION lib/${PROJECT_NAME})
install(DIRECTORY include/ DESTINATION include)
install(DIRECTORY params config DESTINATION share/${PROJECT_NAME})

ament_export_targets(export_pmm_planner HAS_LIBRARY_TARGET)
ament_export_include_directories(include ${EIGEN3_INCLUDE_DIRS})
ament_export_libraries(pmm_planner)
ament_export_dependencies(Eigen3 laser_msgs nav_msgs Threads yaml-cpp)
ament_package()

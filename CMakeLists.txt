cmake_minimum_required(VERSION 3.5)
project(patchworkpp)

# Only the Patchwork++ library is built here. Upstream's own cpp/CMakeLists.txt
# additionally downloads and patches its own Eigen, and builds classic Patchwork
# (the sole consumer of the optional TBB dependency), the Python bindings, the
# ROS wrapper and the Open3D examples. None of that is wanted on the robot.
find_package(ament_cmake REQUIRED)
find_package(Eigen3 REQUIRED)

add_library(patchworkpp STATIC
  cpp/common/src/plane_fit.cpp
  cpp/patchworkpp/src/patchworkpp.cpp
)

target_compile_features(patchworkpp PUBLIC cxx_std_20)
set_target_properties(patchworkpp PROPERTIES POSITION_INDEPENDENT_CODE ON)

target_include_directories(patchworkpp PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/cpp/common/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/cpp/patchworkpp/include>
  $<INSTALL_INTERFACE:include>
)

target_link_libraries(patchworkpp PUBLIC Eigen3::Eigen)

# Inherited code, built without our warning flags: holding it to rules it was
# never written against would mean rewriting it rather than maintaining it.
target_compile_options(patchworkpp PRIVATE -w)

install(
  DIRECTORY cpp/common/include/patchwork cpp/patchworkpp/include/patchwork
  DESTINATION include
)

install(
  TARGETS patchworkpp
  EXPORT export_patchworkpp
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  INCLUDES DESTINATION include
)

ament_export_targets(export_patchworkpp HAS_LIBRARY_TARGET)
ament_export_dependencies(Eigen3)
ament_package()

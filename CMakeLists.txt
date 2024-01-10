# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not
# use this file except in compliance with the License. You may obtain a copy of
# the License at
#
# https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations under
# the License.

cmake_minimum_required(VERSION 3.14)

project(osqp-cpp VERSION 0.5.1)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Options.
option(OSQP_CPP_INSTALL "Install CMake targets during install step." ON)

include(FetchContent)

# ABSL
find_package(absl REQUIRED)

# EIGEN
find_package(Eigen3 3.3.7 REQUIRED NO_MODULE)

# OSQP
find_package(osqp REQUIRED)

message(STATUS "osqp-cpp: Adding osqp-cpp library...")
add_library(osqp-cpp src/osqp++.cc)
add_library(osqp-cpp::osqp-cpp ALIAS osqp-cpp)
target_link_libraries(
  osqp-cpp
  PUBLIC absl::strings absl::status absl::statusor Eigen3::Eigen
  PRIVATE osqp::osqpstatic ${CMAKE_DL_LIBS})
target_include_directories(
  osqp-cpp PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                  $<INSTALL_INTERFACE:include>)
message(STATUS "osqp-cpp: Added osqp-cpp library.")

# Install.
if(OSQP_CPP_INSTALL)
  include(GNUInstallDirs)

  # Install library headers.
  install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

  # Install library binaries.
  install(
    TARGETS osqp-cpp
    EXPORT ${PROJECT_NAME}Targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

  include(CMakePackageConfigHelpers)

  configure_package_config_file(
    osqp-cppConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/osqp-cpp)

  write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY SameMajorVersion)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
                ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
          DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/osqp-cpp)

  # Perform installation.
  install(
    EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE osqp-cpp::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/osqp-cpp)
endif()

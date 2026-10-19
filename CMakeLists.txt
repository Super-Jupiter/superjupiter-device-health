cmake_minimum_required(VERSION 3.16)
project(SuperJupiterDeviceHealth VERSION 0.1.1 LANGUAGES CXX)

set(_superjupiter_device_health_top_level OFF)
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(_superjupiter_device_health_top_level ON)
endif()
option(SUPERJUPITER_DEVICE_HEALTH_BUILD_TESTS "Build DeviceHealth tests" ${_superjupiter_device_health_top_level})
option(SUPERJUPITER_DEVICE_HEALTH_BUILD_EXAMPLES "Build DeviceHealth host example" ${_superjupiter_device_health_top_level})
option(SUPERJUPITER_DEVICE_HEALTH_INSTALL "Generate DeviceHealth install rules" ${_superjupiter_device_health_top_level})

add_library(SuperJupiterDeviceHealth INTERFACE)
add_library(SuperJupiter::DeviceHealth ALIAS SuperJupiterDeviceHealth)
set_target_properties(SuperJupiterDeviceHealth PROPERTIES EXPORT_NAME DeviceHealth)
target_include_directories(SuperJupiterDeviceHealth INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include>)
target_compile_features(SuperJupiterDeviceHealth INTERFACE cxx_std_11)

if(SUPERJUPITER_DEVICE_HEALTH_BUILD_TESTS)
  enable_testing()
  add_executable(superjupiter-device-health-review tests/review_regressions.cpp)
  target_link_libraries(superjupiter-device-health-review PRIVATE SuperJupiter::DeviceHealth)
  add_test(NAME superjupiter-device-health-review COMMAND superjupiter-device-health-review)
  add_executable(superjupiter-device-health-tests tests/test.cpp)
  target_link_libraries(superjupiter-device-health-tests PRIVATE SuperJupiter::DeviceHealth)
  add_test(NAME superjupiter-device-health-tests COMMAND superjupiter-device-health-tests)
  add_executable(superjupiter-device-health-namespace-test tests/namespace_isolation.cpp)
  target_link_libraries(superjupiter-device-health-namespace-test PRIVATE SuperJupiter::DeviceHealth)
  add_test(NAME superjupiter-device-health-namespace-test COMMAND superjupiter-device-health-namespace-test)
endif()
if(SUPERJUPITER_DEVICE_HEALTH_BUILD_EXAMPLES)
  add_executable(superjupiter-device-health-replay tools/replay.cpp)
  target_link_libraries(superjupiter-device-health-replay PRIVATE SuperJupiter::DeviceHealth)
endif()
if(SUPERJUPITER_DEVICE_HEALTH_INSTALL)
  include(CMakePackageConfigHelpers)
  set(_superjupiter_device_health_package_dir lib/cmake/SuperJupiterDeviceHealth)
  install(TARGETS SuperJupiterDeviceHealth EXPORT SuperJupiterDeviceHealthTargets)
  install(FILES src/SuperJupiterDeviceHealth.h DESTINATION include)
  install(EXPORT SuperJupiterDeviceHealthTargets NAMESPACE SuperJupiter::
    DESTINATION ${_superjupiter_device_health_package_dir})
  configure_package_config_file(cmake/SuperJupiterDeviceHealthConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterDeviceHealthConfig.cmake
    INSTALL_DESTINATION ${_superjupiter_device_health_package_dir})
  write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterDeviceHealthConfigVersion.cmake
    VERSION ${PROJECT_VERSION} COMPATIBILITY SameMinorVersion ARCH_INDEPENDENT)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterDeviceHealthConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterDeviceHealthConfigVersion.cmake
    DESTINATION ${_superjupiter_device_health_package_dir})
  install(FILES LICENSE DESTINATION share/licenses/SuperJupiterDeviceHealth)
endif()

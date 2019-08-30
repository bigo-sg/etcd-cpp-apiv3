cmake_minimum_required (VERSION 3.1.3 FATAL_ERROR)
project (etcd-cpp-api)

find_package(cpprestsdk REQUIRED)
find_package(Boost REQUIRED COMPONENTS system thread locale random)
find_package(c-ares REQUIRED)
find_package(gRPC REQUIRED)

set (etcd-cpp-api_VERSION_MAJOR 0)
set (etcd-cpp-api_VERSION_MINOR 1)

include_directories(SYSTEM ${CPPREST_INCLUDE_DIR} ${Boost_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
OPTION(WITH_TESTING OFF "whether open unit testing")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wpedantic -std=c++11")

add_subdirectory(src)

if (WITH_TESTING)
    enable_testing()
    add_subdirectory(tst)
endif()

set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_PREFIX}/lib/cmake/etcd-cpp-api)

# install all targets
install(
    EXPORT etcd-cpp-api
    FILE etcd-cpp-apiTargets.cmake
    NAMESPACE etcd-cpp-api::
    DESTINATION ${INSTALL_CONFIGDIR}
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/etcd-cpp-apiConfigVersion.cmake
    VERSION "1.0"
    COMPATIBILITY AnyNewerVersion
)

# configure package file
configure_package_config_file(
    ${PROJECT_SOURCE_DIR}/cmake/etcd-cpp-apiConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/etcd-cpp-apiConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

# now install all helper config files
install(
    FILES
    ${CMAKE_CURRENT_BINARY_DIR}/etcd-cpp-apiConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/etcd-cpp-apiConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

export(PACKAGE etcd-cpp-api)

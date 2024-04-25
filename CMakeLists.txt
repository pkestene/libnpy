cmake_minimum_required(VERSION 3.10)

if(NOT DEFINED PROJECT_NAME)
  set(LIBNPY_IS_TOP_LEVEL ON)
else()
  set(LIBNPY_IS_TOP_LEVEL OFF)
endif()

project(libnpy
  VERSION 1.0.1
  DESCRIPTION "A simple C++ library for reading and writing of numpy's .npy files."
  HOMEPAGE_URL "https://github.com/llohse/libnpy"
  LANGUAGES C CXX)

include(cmake/prevent_build_in_source.cmake)
include(cmake/init_build_type.cmake)

option(LIBNPY_INSTALL "Include an install target" ON)
option(LIBNPY_BUILD_TESTS "Build tests" ON)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

add_library(libnpy INTERFACE)
add_library(libnpy::libnpy ALIAS libnpy)

target_include_directories(libnpy INTERFACE
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>)

if(LIBNPY_BUILD_TESTS AND LIBNPY_IS_TOP_LEVEL)
  include(CTest)
  add_subdirectory(tests)
endif()

if(LIBNPY_INSTALL AND LIBNPY_IS_TOP_LEVEL)
  install(TARGETS libnpy EXPORT libnpyConfig)
  install(EXPORT libnpyConfig
    NAMESPACE libnpy::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
  install(FILES ${CMAKE_CURRENT_LIST_DIR}/include/npy.hpp
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})


  set(CONFIG_FILE_NAME_WITHOUT_EXT "${PROJECT_NAME}Config")
  set(CMAKE_CONFIG_FILE_BASENAME "${CMAKE_CURRENT_BINARY_DIR}/${CONFIG_FILE_NAME_WITHOUT_EXT}")
  set(CMAKE_CONFIG_VERSION_FILE_NAME "${CMAKE_CONFIG_FILE_BASENAME}-version.cmake")
  set(CMAKE_CONFIG_FILE_NAME "${CMAKE_CONFIG_FILE_BASENAME}.cmake")

  if(${CMAKE_VERSION} VERSION_GREATER "3.14")
  	set(OPTIONAL_ARCH_INDEPENDENT "ARCH_INDEPENDENT")
  endif()

  write_basic_package_version_file("${CMAKE_CONFIG_VERSION_FILE_NAME}"
    COMPATIBILITY ExactVersion
    ${OPTIONAL_ARCH_INDEPENDENT}
    )

  export(EXPORT libnpyConfig
    NAMESPACE libnpy::)

  install(FILES "${CMAKE_CONFIG_VERSION_FILE_NAME}"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

endif()

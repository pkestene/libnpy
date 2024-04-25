cmake_minimum_required(VERSION 3.10)

if (NOT DEFINED CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

# avoid conflicts between
# -Wpedantic being enabled and -std=c++14 and -std=gnu++14
set(CMAKE_CXX_EXTENSIONS OFF)

project(libnpy-tests)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

find_package(Catch2 REQUIRED)

find_package(Python3 REQUIRED)
set(PYTHON_EXECUTABLE ${Python3_EXECUTABLE})

add_custom_target(gen-data
    COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_SOURCE_DIR}/tests/createnpy.py
    COMMENT "Generate npy data files.")

add_executable(test-read test-read.cpp)
target_link_libraries(test-read PRIVATE libnpy::libnpy Catch2::Catch2WithMain)
add_dependencies(test-read gen-data)

add_executable(test-write test-write.cpp)
target_link_libraries(test-write PRIVATE libnpy::libnpy Catch2::Catch2WithMain)
add_dependencies(test-write gen-data)

include(CTest)
include(Catch)
catch_discover_tests(test-read test-write)

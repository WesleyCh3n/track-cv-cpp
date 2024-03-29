cmake_minimum_required(VERSION 3.18)

project(sort-cpp)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD 17)

option(BUILD_EXAMPLES "Build examples" ON)

#==============================================================================
# Third Party Dependencies
#==============================================================================
find_package(OpenCV REQUIRED)

#==============================================================================
# Build Libraries
#==============================================================================
include(GNUInstallDirs)
add_library(sort
  src/sort.cpp
  src/hungarian.cpp
)
target_link_libraries(sort PUBLIC
  ${OpenCV_LIBS}
)
target_include_directories(sort PUBLIC
  ${OpenCV_INCLUDE_DIRS}
)
target_include_directories(sort PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
set(SORT_PUBLIC_HEADERS
    "include/track/sort.hpp"
    "include/track/hungarian.hpp"
    "include/track/kalman.hpp"
)
set_target_properties(sort PROPERTIES
  PUBLIC_HEADER "${SORT_PUBLIC_HEADERS}")
add_library(track::sort ALIAS sort)

include(CMakePackageConfigHelpers)
list(APPEND INSTALL_TARGETS
 sort
)
set(CONFIG_EXPORT_NAME track-config)
set(CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/track)
set(PROJECT_CONFIG ${CMAKE_CURRENT_BINARY_DIR}/${CONFIG_EXPORT_NAME}.cmake)

install(TARGETS ${INSTALL_TARGETS} EXPORT ${CONFIG_EXPORT_NAME}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/track)
export(TARGETS ${INSTALL_TARGETS}
       NAMESPACE track::
       FILE ${PROJECT_CONFIG})
install(EXPORT ${CONFIG_EXPORT_NAME}
        DESTINATION "${CMAKE_DIR}"
        NAMESPACE track::
)

#==============================================================================
# Build Examples
#==============================================================================
if(BUILD_EXAMPLES)
  file(
  DOWNLOAD
  https://github.com/cpm-cmake/CPM.cmake/releases/download/v0.38.3/CPM.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake
  EXPECTED_HASH SHA256=cc155ce02e7945e7b8967ddfaff0b050e958a723ef7aad3766d368940cb15494
  )
  include(${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake)
  CPMAddPackage("gh:fmtlib/fmt#10.2.1")
  add_executable(main
    ${CMAKE_CURRENT_SOURCE_DIR}/example/main.cpp
  )
  target_link_libraries(main PRIVATE
    track::sort
    fmt::fmt
  )
endif()

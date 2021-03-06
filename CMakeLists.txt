cmake_minimum_required(VERSION 2.8)

project(lemon_ros)

ENABLE_LANGUAGE(CXX)

include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_CXX11)
check_cxx_compiler_flag("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()


find_package(catkin REQUIRED COMPONENTS
  roslib
  roscpp
  rospy
  std_msgs
  geometry_msgs
  message_generation
)

# add_message_files(
#   FILES
# 
# )
# 
# add_service_files(
#   FILES
# )

# generate_messages(DEPENDENCIES std_msgs geometry_msgs)

catkin_package(
  INCLUDE_DIRS include/ src/
  LIBRARIES ${PROJECT_NAME} lemon
#  CATKIN_DEPENDS roscpp
   #DEPENDS
   )
include_directories(include ${catkin_INCLUDE_DIRS})
link_directories(${catkin_LIBRARY_DIRS})
include_directories(src/)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/src)

add_subdirectory(src/lemon)

add_library(${PROJECT_NAME} SHARED src/geometry_tools.cpp)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)

install(TARGETS ${PROJECT_NAME}
         EXPORT ${PROJECT_NAME}
#         ARCHIVE DESTINATION "${CATKIN_PACKAGE_BIN_DESTINATION}" COMPONENT lib
#         RUNTIME DESTINATION "${CATKIN_PACKAGE_BIN_DESTINATION}" COMPONENT bin
         LIBRARY DESTINATION "${CATKIN_PACKAGE_LIB_DESTINATION}" COMPONENT shlib
)

INSTALL(
  DIRECTORY ./lemon lemon/bits lemon/concepts
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}/lemon
  COMPONENT headers
  FILES_MATCHING PATTERN "*.h"
)

INSTALL(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/lemon/config.h
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}/lemon
  COMPONENT headers
)

INSTALL(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/lemon.pc
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}/pkgconfig
)

add_custom_command(TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND echo 'Installing lemon config.h...'
#     COMMAND make install
#     WORKING_DIRECTORY ${CATKIN_PACKAGE_BIN_DESTINATION}/../../
    COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/src/lemon/config.h ${CMAKE_CURRENT_SOURCE_DIR}/src/lemon/config.h
)

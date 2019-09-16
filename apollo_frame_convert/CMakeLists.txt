cmake_minimum_required(VERSION 2.8.3)
project(apollo_frame_convert)

set(CMAKE_CXX_STANDARD 11)

#  enable tests
set(CATKIN_ENABLE_TESTING ON)

find_package(catkin REQUIRED)
list(APPEND LINKER_LIBS ${catkin_LIBRARIES})

find_package(Boost COMPONENTS filesystem REQUIRED)
list(APPEND LINKER_LIBS ${Boost_FILESYSTEM_LIBRARIES})

find_package(Eigen3 QUIET)
if(NOT EIGEN3_FOUND)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(EIGEN3 REQUIRED eigen3)
    set(EIGEN3_INCLUDE_DIR ${EIGEN3_INCLUDE_DIRS})
endif()
list(APPEND LINKER_LIBS ${EIGEN3_LIBRARIES})

include("cmake/FindProtobuf.cmake")
list(APPEND LINKER_LIBS ${PROTOBUF_LIBRARIES})

include("cmake/FindGlog.cmake")
list(APPEND LINKER_LIBS ${GLOG_LIBRARIES})

#to other libs
catkin_package(
        ## export these folder sturcture in the package root folder to other catkin packages
        INCLUDE_DIRS include
        LIBRARIES ${PROJECT_NAME})  # we are going to wrap all the algorithms to one library named by the module_name )

include_directories(
        include
        ${catkin_INCLUDE_DIRS}
        ${GLOG_INCLUDE_DIRS}
        ${PROTOBUF_INCLUDE_DIRS}
        ${EIGEN3_INCLUDE_DIRS}
        ${Boost_FILESYSTEM_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}
        src/cartesian_frenet_conversion.cpp
        src/math_utils.cpp
        src/string_util.cpp
        src/vec2d.cpp)

target_link_libraries(${PROJECT_NAME}
        ${LINKER_LIBS})

if (CATKIN_ENABLE_TESTING)
    catkin_add_gtest(${PROJECT_NAME}_test
            tests/convert_test.cpp
            tests/cartesian_frenet_conversion_test.cpp)

    target_include_directories(${PROJECT_NAME}_test
            PUBLIC
            $<BUILD_INTERFACE:${CATKIN_DEVEL_PREFIX}/include/${PROJECT_NAME}>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}>
            )

        message(STATUS "CMAKE_CURRENT_BINARY_DIR: " ${CMAKE_CURRENT_BINARY_DIR})

    target_link_libraries(${PROJECT_NAME}_test
            ${catkin_LIBRARIES}
            ${PROJECT_NAME}
            )
endif (CATKIN_ENABLE_TESTING)
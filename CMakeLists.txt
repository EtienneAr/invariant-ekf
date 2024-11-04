# Specify the minimum version for CMake
cmake_minimum_required(VERSION 3.15.0)
# Project's name
project(inekf)

option(USE_CPP11 "Use flag -std=c++11" ON)
option(USE_MUTEX "Use mutex in code" OFF)

message("USE_CPP11: " ${USE_CPP11})
message("USE_MUTEX: " ${USE_MUTEX})

if (USE_MUTEX AND NOT USE_CPP11)
    message(FATAL_ERROR "You cannot have USE_CPP11 = OFF AND USE_MUTEX = ON")
endif (USE_MUTEX AND NOT USE_CPP11)

# Set compiler flags
if (USE_CPP11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif(USE_CPP11)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -DEIGEN_NO_DEBUG -march=native -Wl,--no-as-needed")
if (USE_MUTEX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DINEKF_USE_MUTEX=true")
else (USE_MUTEX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DINEKF_USE_MUTEX=false")
endif(USE_MUTEX)

message("CMAKE_CXX_FLAGS: " ${CMAKE_CXX_FLAGS})
unset(USE_CPP11)
unset(USE_MUTEX)

############################################
#  The following folders will be included  #
############################################
# Threading
find_package (Threads)

# Boost
find_package(Boost 1.57 REQUIRED COMPONENTS system)
include_directories(${Boost_INCLUDE_DIR})
message("Boost_INCLUDE_DIR: " ${Boost_INCLUDE_DIR})

# Eigen
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})
message("EIGEN3_INCLUDE_DIR: " ${EIGEN3_INCLUDE_DIR})

# Adding all classes
file(GLOB src_files
    "${PROJECT_SOURCE_DIR}/src/*.cpp"
)

##################
# Create Library #
##################

add_library(inekf SHARED ${src_files})
target_include_directories(inekf PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

include(GNUInstallDirs)

install(
    TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-config
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    INCLUDES include
)

install(
    EXPORT ${PROJECT_NAME}-config
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION cmake/${PROJECT_NAME})

install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

######################
#  Add Execuatables  #
######################
add_executable(landmarks ${PROJECT_SOURCE_DIR}/src/examples/landmarks.cpp)
add_executable(kinematics ${PROJECT_SOURCE_DIR}/src/examples/kinematics.cpp)
add_executable(propagation_speed ${PROJECT_SOURCE_DIR}/src/tests/propagation_speed.cpp)
add_executable(correction_speed ${PROJECT_SOURCE_DIR}/src/tests/correction_speed.cpp)

target_link_libraries(landmarks ${Boost_LIBRARIES} inekf)
target_link_libraries(kinematics ${Boost_LIBRARIES} inekf)
target_link_libraries(propagation_speed ${Boost_LIBRARIES} inekf)
target_link_libraries(correction_speed ${Boost_LIBRARIES} inekf)

install(TARGETS
    landmarks
    kinematics
    propagation_speed
    correction_speed
  DESTINATION ${CMAKE_INSTALL_EXECDIR})

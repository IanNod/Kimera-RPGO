cmake_minimum_required (VERSION 3.5)
project(KimeraRPGO VERSION 1.0 LANGUAGES CXX)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

IF(APPLE)
    # Fix linking on 10.14+. See https://stackoverflow.com/questions/54068035
    LINK_DIRECTORIES(/usr/local/lib)
ENDIF()

###########################################################################
# Find GTSAM
find_package(GTSAM REQUIRED)
if (NOT GTSAM_FOUND)
  message(FATAL_ERROR "This program requires the GTSAM library.")
endif(NOT GTSAM_FOUND)

###########################################################################
# Find Boost
find_package(Boost REQUIRED COMPONENTS filesystem regex timer date_time program_options)
if(NOT TARGET Boost::boost)
  add_library(Boost::boost INTERFACE IMPORTED)
  set_target_properties(Boost::boost PROPERTIES
  INTERFACE_LINK_LIBRARIES "${Boost_LIBRARIES}"
  INTERFACE_INCLUDE_DIRECTORIES "${Boost_INCLUDE_DIRS}")
endif()

###########################################################################
# Compile
add_library(KimeraRPGO SHARED
  KimeraRPGO/logger.h
  KimeraRPGO/GenericSolver.cpp
  KimeraRPGO/RobustSolver.cpp
  KimeraRPGO/SolverParams.h )

# Add source code for max clique finder
include(KimeraRPGO/max_clique_finder/CMakeLists.txt)
# Add source for outlier rejection stuff
include(KimeraRPGO/outlier/CMakeLists.txt)
# Add source for utils
include(KimeraRPGO/utils/CMakeLists.txt)

target_link_libraries(KimeraRPGO
  PUBLIC
    Boost::boost
    gtsam
    gtsam_unstable
)

target_include_directories(KimeraRPGO
  PUBLIC
    ${GTSAM_INCLUDE_DIR}
    # ${CMAKE_CURRENT_SOURCE_DIR}/gtsam
)

###########################################################################
# Define executables
add_executable(RpgoReadG2o examples/RpgoReadG2o.cpp)
target_link_libraries(RpgoReadG2o KimeraRPGO)

###########################################################################


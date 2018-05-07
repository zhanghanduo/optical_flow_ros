cmake_minimum_required( VERSION 2.8.3 )
if( POLICY CMP0048 )
  # Silence CMP0048 warning about missing project VERSION.
  cmake_policy(SET CMP0048 NEW)
endif()
if( POLICY CMP0042 )
  cmake_policy(SET CMP0042 NEW)
endif()
project( OpticalFlow )
set(version_major 1)
set(version_minor 1)
set(version_patch 1)
set(version "${version_major}.${version_minor}.${version_patch}")
set(package-contact "hzhang032@e.ntu.edu.sg")

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpic")
SET(flow_C_FLAGS)
SET(flow_CXX_FLAGS)

# default built type
IF(NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE(STATUS "BUILD TYPE:" ${CMAKE_BUILD_TYPE})

SET (FLOW_LIB_TYPE SHARED)

# Set the output directory for the build executables and libraries
IF(WIN32)
    SET(OpticalFlow_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin CACHE PATH "Target for the libraries")
ELSE(WIN32)
    SET(OpticalFlow_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib CACHE PATH "Target for the libraries")
ENDIF(WIN32)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OpticalFlow_LIBRARY_OUTPUT_DIRECTORY})
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OpticalFlow_LIBRARY_OUTPUT_DIRECTORY})
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OpticalFlow_RUNTIME_OUTPUT_DIRECTORY})

# Compiler specific options for gcc
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -march=native")
SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -march=native")

# activate warnings !!!
SET(flow_C_FLAGS "${flow_C_FLAGS} -Wall -W -fpic")
SET(flow_CXX_FLAGS "${flow_CXX_FLAGS} -Wall -W -fpic")

# specifying compiler flags
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${flow_CXX_FLAGS}")
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${flow_C_FLAGS}")

## Require C++14 support, since we use specific features
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(${PROJECT_NAME}_INCLUDE_DIRS
    ${PROJECT_SOURCE_DIR}/include
    CACHE INTERNAL "${PROJECT_NAME}: Include Directories" FORCE)

set(OpenCV_DIR /home/hd/softwares/opencv-3.3.0/release)

find_package(OpenCV 3.1 REQUIRED)
OPTION(OPENCV "ON for OpenCV" ON) # -DOPENCV=OFF

if(OpenCV_FOUND AND OPENCV)
    message(STATUS "Building ${PROJECT_NAME} with OpenCV")

    set(OpticalFlow_INCLUDE_DIRS
      ${CMAKE_CURRENT_SOURCE_DIR}/include
      ${OpenCV_INCLUDE_DIRS}
    )

    include_directories(OpticalFlow_INCLUDE_DIRS)

    add_library( OpticalFlow ${FLOW_LIB_TYPE}
        src/optical_flow.cpp
#       src/px4flow.cpp
#     	src/flow_px4.cpp
    	src/flow_opencv.cpp
        src/trackFeatures.cpp
    )

    target_link_libraries( OpticalFlow
      ${OpenCV_LIBS}
    )
else()

    message(FATAL_ERROR "Exit compiling without OpenCV...")

endif()

target_include_directories(OpticalFlow PUBLIC include)

# Generate the Config file from the .in file, doing @VAR@ substitution only
#configure_file(OpticalFlowConfig.cmake.in FindOpticalFlow.cmake @ONLY)

# INSTALL STEPS
INSTALL(
  TARGETS OpticalFlow
  RUNTIME DESTINATION lib
  LIBRARY DESTINATION lib)

INSTALL(
  DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
  DESTINATION include)
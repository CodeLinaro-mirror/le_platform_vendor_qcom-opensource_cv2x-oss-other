cmake_minimum_required(VERSION 2.8)

project("Cv2xRemoteAPI-Gen")
set(PRJ_NAME Cv2xRemoteAPI)

set(CMAKE_VERBOSE_MAKEFILE on)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE BOTH)
set(CMAKE_FIND_DEBUG_MODE TRUE)

OPTION(USE_FILE "Set to OFF to disable file logging" OFF )
message(STATUS "USE_FILE is set to value: ${USE_FILE}")

OPTION(USE_CONSOLE "Set to OFF to disable console logging" OFF )
message(STATUS "USE_CONSOLE is set to value: ${USE_CONSOLE}")

IF(USE_FILE)
  add_definitions(-DUSE_FILE)
ENDIF(USE_FILE)
IF(USE_CONSOLE)
  add_definitions(-DUSE_CONSOLE)
ENDIF(USE_CONSOLE)

SET(MAX_LOG_LEVEL "DEBUG" CACHE STRING "maximum log level")
message(STATUS "MAX_LOG_LEVEL is set to value: ${MAX_LOG_LEVEL}")
add_definitions(-DCOMMONAPI_LOGLEVEL=COMMONAPI_LOGLEVEL_${MAX_LOG_LEVEL})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pthread -O0 -std=c++11 -D_GLIBCXX_USE_NANOSLEEP")

message(STATUS "Compiler options: ${CMAKE_CXX_FLAGS}")

if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
       "Choose the type of build, options are: Debug Release." FORCE)
endif(NOT CMAKE_BUILD_TYPE)
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

OPTION(USE_INSTALLED_COMMONAPI "Set to OFF to use the local (build tree) version of CommonAPI" ON)
message(STATUS "USE_INSTALLED_COMMONAPI is set to value: ${USE_INSTALLED_COMMONAPI}")

if ("${USE_INSTALLED_COMMONAPI}" STREQUAL "ON")
        FIND_PACKAGE(CommonAPI 3.2.0 REQUIRED CONFIG NO_CMAKE_PACKAGE_REGISTRY)
else()
        FIND_PACKAGE(CommonAPI 3.2.0 REQUIRED CONFIG NO_SYSTEM_ENVIRONMENT_PATH NO_CMAKE_SYSTEM_PATH)
endif()

message(STATUS "CommonAPI_CONSIDERED_CONFIGS: ${CommonAPI_CONSIDERED_CONFIGS}")
message(STATUS "COMMONAPI_INCLUDE_DIRS: ${COMMONAPI_INCLUDE_DIRS}")

# CommonAPI
include(FindPkgConfig)
###############################################################################

# SOME/IP
find_package (CommonAPI-SomeIP 3.2.0 REQUIRED)
find_package (vsomeip3 3.1.0 REQUIRED)

# Source Files
set(PRJ_SRC_GEN_COMMONAPI_SOMEIP_PATH ${CMAKE_CURRENT_SOURCE_DIR}/someip/v1/commonapi/telux)



pkg_check_modules(GLIB REQUIRED glib-2.0)
add_compile_options(${GLIB_CFLAGS})

#Fast-DDS and Fast-CDR
find_package(fastcdr REQUIRED)
find_package(fastrtps 2.12 REQUIRED)

# Boost
find_package( Boost 1.54 COMPONENTS system thread log REQUIRED )
include_directories( ${Boost_INCLUDE_DIR} )

# SOME/IP library
FILE(GLOB PRJ_SOMEIP_LIB_SRCS ${PRJ_SRC_GEN_COMMONAPI_SOMEIP_PATH}/common/*cpp
                              ${PRJ_SRC_GEN_COMMONAPI_SOMEIP_PATH}/cv2x/*cpp
                              ${PRJ_SRC_GEN_COMMONAPI_SOMEIP_PATH}/cv2x/radio/*cpp
    )
message(STATUS "TELUX_HEADER_PATH: ${TELUX_HEADER_PATH}")
include_directories(
    core
    someip
    dds
    ${COMMONAPI_INCLUDE_DIRS}
    ${VSOMEIP_INCLUDE_DIRS}
    ${TELUX_HEADER_PATH}
)

set(LINK_LIBRARIES -Wl,--as-needed CommonAPI)

# Build SOME/IP library
add_library (${PRJ_NAME}-someip SHARED ${PRJ_SOMEIP_LIB_SRCS})
target_link_libraries(${PRJ_NAME}-someip CommonAPI-SomeIP)

#cv2x dds library
set(CV2X_DATA_LIB_GEN Cv2xDataGen)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/dds)

install(TARGETS ${PRJ_NAME}-someip
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
#install headers
file(GLOB_RECURSE CORE_TELUX_COMMON_HEADERS core/v1/commonapi/telux/common/*.hpp)
file(GLOB_RECURSE CORE_TELUX_CV2X_HEADERS core/v1/commonapi/telux/cv2x/*.hpp)
file(GLOB_RECURSE SOMEIP_TELUX_COMMON_HEADERS someip/v1/commonapi/telux/common/*.hpp)
file(GLOB_RECURSE SOMEIP_TELUX_CV2X_HEADERS someip/v1/commonapi/telux/cv2x/*.hpp)
install(FILES ${CORE_TELUX_COMMON_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/cv2x-remote-gen/core/v1/commonapi/telux/common/
    COMPONENT Headers
)
install(FILES ${CORE_TELUX_CV2X_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/cv2x-remote-gen/core/v1/commonapi/telux/cv2x/
    COMPONENT Headers
    )
install(FILES ${SOMEIP_TELUX_COMMON_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/cv2x-remote-gen/someip/v1/commonapi/telux/cv2x/
    COMPONENT Headers
    )
install(FILES ${CORE_TELUX_CV2X_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/cv2x-remote-gen/someip/v1/commonapi/telux/cv2x/
    COMPONENT Headers
    )

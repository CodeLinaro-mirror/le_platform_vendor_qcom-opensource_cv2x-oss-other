cmake_minimum_required(VERSION 3.10)
include(GNUInstallDirs)

project("SplitsCommonAPI-Gen")
set(PRJ_NAME SplitsService)

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
set(PRJ_SRC_GEN_COMMONAPI_SOMEIP_PATH ${CMAKE_CURRENT_SOURCE_DIR}/someip/v1/commonapi/SPLITS/)

pkg_check_modules(GLIB REQUIRED glib-2.0)
add_compile_options(${GLIB_CFLAGS})

# Boost
find_package( Boost 1.54 COMPONENTS system thread log REQUIRED )
include_directories( ${Boost_INCLUDE_DIR} )

# SOME/IP library
FILE(GLOB PRJ_SOMEIP_LIB_SRCS ${PRJ_SRC_GEN_COMMONAPI_SOMEIP_PATH}/*.cpp
    )

include_directories(
    core
    someip
    ${COMMONAPI_INCLUDE_DIRS}
    ${VSOMEIP_INCLUDE_DIRS}
)

set(LINK_LIBRARIES -Wl,--as-needed CommonAPI)

# Build SOME/IP library
add_library (${PRJ_NAME}-someip SHARED ${PRJ_SOMEIP_LIB_SRCS})
target_link_libraries(${PRJ_NAME}-someip CommonAPI-SomeIP)

install(TARGETS ${PRJ_NAME}-someip
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
#install headers
file(GLOB_RECURSE CORE_SPLITS_HEADERS core/v1/commonapi/SPLITS/*.hpp)
file(GLOB_RECURSE SOMEIP_SPLITS_HEADERS someip/v1/commonapi/SPLITS/*.hpp)
file(GLOB_RECURSE CORE_COMMON_HEADERS core/v1/commonapi/telux/common/*.hpp)
file(GLOB_RECURSE SOMEIP_COMMON_HEADERS someip/v1/commonapi/telux/common/*.hpp)
install(FILES ${CORE_SPLITS_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/splits-commonapi-gen/core/v1/commonapi/SPLITS/
    COMPONENT Headers
)
install(FILES ${SOMEIP_SPLITS_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/splits-commonapi-gen/someip/v1/commonapi/SPLITS/
    COMPONENT Headers
)
install(FILES ${CORE_COMMON_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/splits-commonapi-gen/core/v1/commonapi/telux/common/
    COMPONENT Headers
)
install(FILES ${SOMEIP_COMMON_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/splits-commonapi-gen/someip/v1/commonapi/telux/common/
    COMPONENT Headers
)

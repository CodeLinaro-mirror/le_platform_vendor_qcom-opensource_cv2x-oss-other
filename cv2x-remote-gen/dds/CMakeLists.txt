cmake_minimum_required(VERSION 3.10)

project("Cv2xDataGen")
include(GNUInstallDirs)

set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE BOTH)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)

# Find requirements
find_package(fastcdr REQUIRED)
find_package(fastrtps 2.12 REQUIRED)

#include_directories(AFTER ${TELUX_HEADER_PATH})
message(STATUS "Configuring Cv2xDataGen...")
add_library(${CV2X_DATA_LIB_GEN} SHARED Cv2xData.cxx
        Cv2xDataPubSubTypes.cxx
)
set_target_properties(${CV2X_DATA_LIB_GEN} PROPERTIES PUBLIC_HEADER "Cv2xDataPubSubTypes.h;Cv2xData.h")
target_link_libraries(${CV2X_DATA_LIB_GEN} fastcdr fastrtps)

install(TARGETS ${CV2X_DATA_LIB_GEN}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

# Copyright (C) 2018-2021 Intel Corporation
# SPDX-License-Identifier: Apache-2.0
# With additions of Luxonis

cmake_minimum_required(VERSION 3.2)
string(ASCII 27 Esc)
string(ASCII 27 Esc)

set(CYN "${Esc}[0;36m")
set(NOC  "${Esc}[m")
set(CYN "${Esc}[0;36m")
set(NOC  "${Esc}[m")

function(show_message arg_name arg_value)
    message("${CYN}${arg_name}:${NOC} ${arg_value}")
endfunction()


### Constants
set(TARGET_NAME "XLink")
set(TARGET_PUBLIC_NAME "XLinkPublic")

# Create project
project(${TARGET_NAME} LANGUAGES C CXX)

set(PROJECT_EXPORT_GROUP "${PROJECT_NAME}Targets")
message(STATUS "${CYN}PROJECT_EXPORT_GROUP:${NOC} ${PROJECT_EXPORT_GROUP}")

# Set default installation directory -- we should use the default one, instead of revising it
# where is "CMAKE_INSTALL_PREFIX" used?
message(STATUS "${CYN}CMAKE_BINARY_DIR:${NOC} ${CMAKE_BINARY_DIR}")
message(STATUS "${CYN}CMAKE_INSTALL_PREFIX:${NOC} ${CMAKE_INSTALL_PREFIX}")

include(cmake/XLink.cmake)
include(cmake/Flags.cmake)

# Create library target
add_library(${TARGET_NAME} ${XLINK_SOURCES})
add_flag_source(src/shared/XLinkDevice.c "-Werror=switch-enum")
get_cmake_property(is_multi_config GENERATOR_IS_MULTI_CONFIG)
if(is_multi_config)
    set_target_properties(${TARGET_NAME} PROPERTIES DEBUG_POSTFIX "d")
endif()

# Add dependencies
include(cmake/XLinkDependencies.cmake)

# Link to CMake libusb
target_link_libraries(${TARGET_NAME} PRIVATE luxonis-usb-1.0)

find_package(Threads REQUIRED)
target_link_libraries(${TARGET_NAME} PRIVATE Threads::Threads)

# Create public "interface" library (no linking)
add_library(${TARGET_PUBLIC_NAME} INTERFACE)
target_include_directories(${TARGET_PUBLIC_NAME} INTERFACE
    "$<INSTALL_INTERFACE:include>"
    "$<BUILD_INTERFACE:${XLINK_INCLUDE}>"
)

# Link to headers (public, as itself also needs the headers)
target_link_libraries(${TARGET_NAME} PUBLIC ${TARGET_PUBLIC_NAME})

target_include_directories(${TARGET_NAME}
    PRIVATE
        ${XLINK_PLATFORM_INCLUDE}
        ${XLINK_PRIVATE_INCLUDE}
        ${XLINK_INCLUDE}/XLink
)

target_compile_definitions(${TARGET_NAME}
    PRIVATE
        HAVE_STRUCT_TIMESPEC
        _CRT_SECURE_NO_WARNINGS
        USE_USB_VSC
        USE_TCP_IP
)

if (ENABLE_MYRIAD_NO_BOOT)
    target_compile_definitions(${TARGET_NAME}
        PRIVATE
        NO_BOOT)
endif()

# Set C99 standard
set_property(TARGET ${TARGET_NAME} PROPERTY C_STANDARD 99)
# Set compiler features (c++11), and disables extensions (g++11)
set_property(TARGET ${TARGET_NAME} PROPERTY CXX_STANDARD 11)
set_property(TARGET ${TARGET_NAME} PROPERTY CXX_STANDARD_REQUIRED ON)
set_property(TARGET ${TARGET_NAME} PROPERTY CXX_EXTENSIONS OFF)
# Add interface transitive property (C++11) to public library
if(${CMAKE_VERSION} VERSION_LESS "3.8.0")
    target_compile_features(${TARGET_PUBLIC_NAME} INTERFACE cxx_range_for)
else()
    target_compile_features(${TARGET_PUBLIC_NAME} INTERFACE cxx_std_11)
endif()

# Add flags
add_default_flags(${TARGET_NAME})

# Examples
add_subdirectory(examples)

# Tests
add_subdirectory(tests)

# Installation
include(GNUInstallDirs)

# Export targets (capability to import current build directory)
export(TARGETS ${TARGET_NAME} ${TARGET_PUBLIC_NAME} FILE "${PROJECT_NAME}Targets.cmake")

# Dependencies file
configure_file("cmake/${PROJECT_NAME}Dependencies.cmake" ${PROJECT_NAME}Dependencies.cmake COPYONLY)

configure_file(cmake/${PROJECT_NAME}Config.cmake.in ${PROJECT_NAME}Config.cmake @ONLY)

# Config for installation
set(XLINK_DEPENDENCIES_INSTALLATION_PATH_REL "./dependencies")
configure_file(cmake/${PROJECT_NAME}Config.cmake.in _install/${PROJECT_NAME}Config.cmake @ONLY)

# Install targets
install(
    TARGETS ${TARGET_NAME} ${TARGET_PUBLIC_NAME}
    EXPORT ${PROJECT_EXPORT_GROUP}
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)
#Install include folder
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# Install export group (information about targets)
install(EXPORT ${PROJECT_EXPORT_GROUP}
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
)
# Install CMake specific files
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/_install/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Dependencies.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
)

cmake_minimum_required(VERSION 3.5)
project(libsafe16 VERSION 1.0.0 LANGUAGES C CXX)

#Make sure that custom modules are found
list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake)


##############################################
# Create target and set properties

add_library(safe16
    src/library.c
)

#Add an alias so that library can be used inside the build tree, e.g. when testing
add_library(SAFE16::safe16 ALIAS safe16)

#Set target properties
target_include_directories(safe16
    PUBLIC
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

target_compile_features(safe16 PRIVATE cxx_auto_type)
target_compile_options(safe16 PRIVATE $<$<CXX_COMPILER_ID:GNU>:
    -Wall
    -Wextra
    -Wduplicated-cond
    -Wduplicated-branches
    -Wlogical-op
    -Wrestrict
    -Wnull-dereference
    -Wjump-misses-init
    -Wdouble-promotion
    -Wshadow
    -fstrict-aliasing
    -Wformat=2
#    -pedantic
>)

##############################################
# Installation instructions

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/SAFE16)

configure_file(src/safe16_version.h.in safe16_version.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

install(TARGETS safe16
    EXPORT safe16-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

#This is required so that the exported target has the name SAFE16 and not safe16
set_target_properties(safe16 PROPERTIES EXPORT_NAME SAFE16)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

#Export the targets to a script
install(EXPORT safe16-targets
  FILE
    SAFE16Targets.cmake
  NAMESPACE
    SAFE16::
  DESTINATION
    ${INSTALL_CONFIGDIR}
)

#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/Safe16ConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/Safe16Config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/Safe16Config.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

#Install the config, configversion and custom find modules
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/Safe16Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/Safe16ConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

export(EXPORT safe16-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/SAFE16Targets.cmake NAMESPACE SAFE16::)

#Register package in user's package registry
export(PACKAGE SAFE16)

add_subdirectory(test)

cmake_minimum_required(VERSION 3.23)
project(cpp-lcurve)

include(ExternalProject)

message("\nCMakeLists.txt for cpp-lcurve")
message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("CMAKE_BINARY_DIR: ${CMAKE_BINARY_DIR}")

set(PACKAGE_NAME lcurve)

############################################################
# Define sources
############################################################

set(SOURCES
    src/numface.cc
    src/set_star_grid.cc
    src/set_star_emission.cc
    src/light_curve_comp.cc
    src/comp_light.cc
    src/trm_lcurve.cc
    src/set_star_continuum.cc
    src/re_scale.cc
    src/set_disc_grid.cc
    src/set_bright_spot_grid.cc
    src/set_disc_continuum.cc
    src/star_eclipse.cc
    src/comp_gravity.cc
    src/comp_radius.cc
    src/lroche.cc
    src/lprofile.cc
)

############################################################
# Define the library
############################################################
add_library(${PACKAGE_NAME} SHARED 
    ${SOURCES}
)

target_include_directories(${PACKAGE_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

############################################################
# Check for and install dependencies
############################################################
# Configure the PLPLOT library
if(NOT DEFINED CMAKE_COMMON)
    set(CMAKE_COMMON ${PROJECT_SOURCE_DIR}/../../cmake_common)
endif()
if(NOT DEFINED PLPLOT_FOUND)
    include("${CMAKE_COMMON}/plplot_install.cmake")
endif()


# TODO: Make this work with the LLVM OpenMP library
# There is the option to use OpenMP for a little bit of parallelism in set_star_grid and light_curve_comp
# However its a pain and not worth the hassle right now
#find_package(OpenMP REQUIRED)

target_include_directories(${PACKAGE_NAME}
    PUBLIC 
        $<BUILD_INTERFACE:${PLPLOT_INCLUDE_DIR}>
)
#target_compile_options(${PACKAGE_NAME} PRIVATE ${OpenMP_CXX_FLAGS})
target_link_libraries(${PACKAGE_NAME} 
    PUBLIC
        roche
        colly
        #llvm-openmp::llvm-openmp
)

############################################################
# Install
############################################################
include("${CMAKE_COMMON}/install_lib.cmake") # Get the generic install function

install_lib(${PACKAGE_NAME})

message("CMakeLists.txt for cpp-lcurve done\n")
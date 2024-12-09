cmake_minimum_required(VERSION 3.23)
project(cpp-lcurve)

include(ExternalProject)

message("CMakeLists.txt for cpp-subs")
message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("CMAKE_BINARY_DIR: ${CMAKE_BINARY_DIR}")

set(PACKAGE_NAME lcurve)

############################################################
# Define sources
############################################################

set(SOURCES
    src/lprofile.cc
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

include("${CMAKE_BINARY_DIR}/Release/generators/conan_toolchain.cmake")

# Configure the PLPLOT library

include("${CMAKE_SOURCE_DIR}/../../cmake_common/plplot_install.cmake")

find_package(SOFA REQUIRED)
find_package(PCRE2 REQUIRED)
find_package(SUBS REQUIRED)
find_package(COLLY REQUIRED)
find_package(BINARY REQUIRED)
find_package(ROCHE REQUIRED)
find_package(OpenMP REQUIRED)

if (NOT SOFA_FOUND)
    message(FATAL_ERROR "SOFA library not found.")
endif()
if (NOT PCRE2_FOUND)
    message(FATAL_ERROR "PCRE2 library not found.")
endif()
if (NOT PLPLOT_FOUND)
    message(FATAL_ERROR "PLPLOT library not found.")
endif()
if (NOT SUBS_FOUND)
    message(FATAL_ERROR "SUBS library not found.")
endif()
if (NOT COLLY_FOUND)
    message(FATAL_ERROR "COLLY library not found.")
endif()
if (NOT BINARY_FOUND)
    message(FATAL_ERROR "BINARY library not found.")
endif()
if (NOT ROCHE_FOUND)
    message(FATAL_ERROR "ROCHE library not found.")
endif()
if (NOT OpenMP_FOUND)
    message(FATAL_ERROR "OpenMP library not found.")
endif()

target_include_directories(${PACKAGE_NAME}
    PUBLIC 
        $<BUILD_INTERFACE:${PLPLOT_INCLUDE_DIR}>
)
target_link_libraries(${PACKAGE_NAME} 
    PUBLIC 
        sofa::sofa 
        pcre2::pcre2
        ${PLPLOT_LIB_DIR}/libplplotcxx.dylib
        subs::subs
        colly::colly
        binary::binary
        roche::roche
        OpenMP::OpenMP_CXX
)


############################################################
# Install
############################################################

# Library
# Note: may not work on windows
# Specify where the library binary will go
install(TARGETS ${PACKAGE_NAME}
    EXPORT ${PACKAGE_NAME}Targets        # Export the library target for package config
    LIBRARY DESTINATION lib              # For shared libraries
    ARCHIVE DESTINATION lib              # For static libraries
    RUNTIME DESTINATION bin              # For executables (if any)
    INCLUDES DESTINATION include         # Where to put public headers
)

# Header files
install(DIRECTORY include/ 
    DESTINATION include
)

# # Create and install CMake package configuration files
include(CMakePackageConfigHelpers)

# # Configure the version file
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PACKAGE_NAME}ConfigVersion.cmake"
    VERSION 0.1
    COMPATIBILITY AnyNewerVersion
)

configure_file(lcurveConfig.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/${PACKAGE_NAME}Config.cmake"
    @ONLY
)

# # Install the config files
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PACKAGE_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PACKAGE_NAME}ConfigVersion.cmake"
    DESTINATION lib/cmake/${PACKAGE_NAME}
)

# # Export the library target so other projects can import it
install(EXPORT ${PACKAGE_NAME}Targets
    FILE ${PACKAGE_NAME}Targets.cmake
    NAMESPACE lcurve::
    DESTINATION lib/cmake/${PACKAGE_NAME}
)

############################################################
# Build executables that link against the lcurve library
############################################################

# Executable sources
add_executable(lprofile src/lprofile.cc)
add_executable(levmarq src/levmarq.cc)
add_executable(lroche src/lroche.cc)
add_executable(lroches src/lroche.cc)  # Assuming lroches is similar to lroche
add_executable(picture src/picture.cc src/pos_disc.cc src/disc_eclipse.cc)
add_executable(rotprof src/rotprof.cc)
add_executable(simplex src/simplex.cc)
add_executable(visualise src/visualise.cc)

find_package(LCURVE REQUIRED)
if (NOT LCURVE_FOUND)
    message(FATAL_ERROR "LCURVE library not found.")
endif()

# Link the library and OpenMP to executables
foreach(EXEC lprofile levmarq lroche lroches picture rotprof simplex visualise)
    target_include_directories(
        ${EXEC} PRIVATE 
        $<BUILD_INTERFACE:${PLPLOT_INCLUDE_DIR}> 
    )
    target_link_libraries(
        ${EXEC} PRIVATE 
        OpenMP::OpenMP_CXX
        ${PLPLOT_LIB_DIR}/libplplotcxx.dylib
        subs::subs
        ${PACKAGE_NAME}::${PACKAGE_NAME}
    )
endforeach()

############################################################
# Install executables
############################################################
# Ensure RPATH includes the library directories
set(CMAKE_BUILD_RPATH "${PLPLOT_LIB_DIR};/usr/local/lib")
set(CMAKE_INSTALL_RPATH "${PLPLOT_LIB_DIR};/usr/local/lib")

# Allow the RPATH to be used in the build tree
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

install(TARGETS lprofile levmarq lroche lroches picture rotprof simplex visualise
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib          # Shared libraries to lib/
    ARCHIVE DESTINATION lib          # Static libraries to lib/
)
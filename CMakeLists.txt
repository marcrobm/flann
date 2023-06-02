cmake_minimum_required(VERSION 2.8.12)

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 NEW)
    cmake_policy(SET CMP0048 NEW)
endif(COMMAND cmake_policy)

project(flann)
string(TOLOWER ${PROJECT_NAME} PROJECT_NAME_LOWER)

include(${PROJECT_SOURCE_DIR}/cmake/flann_utils.cmake)
set(FLANN_VERSION 1.9.2)
DISSECT_VERSION()
GET_OS_INFO()
ENABLE_TESTING()


# detect if using the Clang compiler
if("${CMAKE_C_COMPILER_ID}" MATCHES "Clang")
  set(CMAKE_COMPILER_IS_CLANG 1)
endif ()

if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  set(CMAKE_COMPILER_IS_CLANGXX 1)
endif ()

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# Add an "uninstall" target
CONFIGURE_FILE ("${PROJECT_SOURCE_DIR}/cmake/uninstall_target.cmake.in"
    "${PROJECT_BINARY_DIR}/uninstall_target.cmake" IMMEDIATE @ONLY)
ADD_CUSTOM_TARGET (uninstall_flann "${CMAKE_COMMAND}" -P
    "${PROJECT_BINARY_DIR}/uninstall_target.cmake")

# Set the build type.  Options are:
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries

if (NOT CMAKE_BUILD_TYPE)
    #set(CMAKE_BUILD_TYPE Release)
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)
    #set(CMAKE_BUILD_TYPE Debug)
endif()

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
# set output path for tests
set(TEST_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/test)

option(BUILD_C_BINDINGS "Build C bindings" ON)
option(BUILD_PYTHON_BINDINGS "Build Python bindings" OFF)
option(BUILD_MATLAB_BINDINGS "Build Matlab bindings" OFF)
option(BUILD_CUDA_LIB "Build CUDA library" OFF)
option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_TESTS "Build tests" OFF)
option(BUILD_DOC "Build documentation" OFF)
option(USE_OPENMP "Use OpenMP multi-threading" OFF)
option(USE_MPI "Use MPI" OFF)

if (USE_OPENMP)
    find_package(OpenMP)
    if(OPENMP_FOUND)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
    else()
        message(WARNING "OpenMP NOT found")
    endif()
endif()

include_directories(${PROJECT_SOURCE_DIR}/../lz4/lib)

#set the C/C++ include path to the "include" directory
include_directories(BEFORE ${PROJECT_SOURCE_DIR}/src/cpp)

# install and export variables
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/flann-config-version.cmake")
set(project_config "${generated_dir}/flann-config.cmake")
set(targets_export_name "flann-targets")
set(namespace "flann::")

add_subdirectory( cmake )
add_subdirectory( src )
# CMake configuration file creation
# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure 'flann-config-version.cmake'
# Note: FLANN_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}"
    VERSION ${FLANN_VERSION}
    COMPATIBILITY SameMajorVersion
)

# Configure 'flann-config.cmake'
# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

set(CPACK_GENERATOR "${CPACK_GENERATOR}")
set(CPACK_MONOLITHIC_INSTALL 1)
set(CPACK_SET_DESTDIR ON)
include(InstallRequiredSystemLibraries)
set(CPACK_PACKAGE_CONTACT "Marius Muja")
set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
SET(CPACK_PACKAGE_VERSION ${FLANN_VERSION})
SET(CPACK_PACKAGE_VERSION_MAJOR ${FLANN_VERSION_MAJOR})
SET(CPACK_PACKAGE_VERSION_MINOR ${FLANN_VERSION_MINOR})
SET(CPACK_PACKAGE_VERSION_PATCH ${FLANN_VERSION_PATCH})
include(CPack)


message(STATUS "Install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Building C bindings: ${BUILD_C_BINDINGS}")
message(STATUS "Building examples: ${BUILD_EXAMPLES}")
message(STATUS "Building tests: ${BUILD_TESTS}")
message(STATUS "Building documentation: ${BUILD_DOC}")
message(STATUS "Building python bindings: ${BUILD_PYTHON_BINDINGS}")
message(STATUS "Building matlab bindings: ${BUILD_MATLAB_BINDINGS}")
message(STATUS "Building CUDA library: ${BUILD_CUDA_LIB}")
message(STATUS "Using OpenMP support: ${USE_OPENMP}")
message(STATUS "Using MPI support: ${USE_MPI}")
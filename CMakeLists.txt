cmake_minimum_required(VERSION 3.10)
project(optix_helpers VERSION 0.1 LANGUAGES CUDA CXX)

find_path(OPTIX_PREFIX "include/optix.h"
    HINTS ${OPTIX_PATH}
)
find_path(OPTIX_INCLUDE_DIR "optix.h"
    HINTS ${OPTIX_PREFIX}/include
)
find_library(optix_LIBRARY "optix"
    HINTS ${OPTIX_PREFIX}
    PATH_SUFFIXES lib lib64
)
find_library(optixu_LIBRARY "optixu"
    HINTS ${OPTIX_PREFIX}
    PATH_SUFFIXES lib lib64
)
find_library(optix_prime_LIBRARY "optix_prime"
    HINTS ${OPTIX_PREFIX}
    PATH_SUFFIXES lib lib64
)
if(${OPTIX_PREFIX} STREQUAL "OPTIX_PREFIX-NOTFOUND")
    message(FATAL_ERROR "Could not find OptiX SDK inside \"${OPTIX_PATH}\" folder. Please check the -DOPTIX_PATH option.")
endif()
message(STATUS "Found OptiX           : ${OPTIX_PREFIX}")
message(STATUS "OptiX include dirs    : ${OPTIX_INCLUDE_DIR}")
message(STATUS "optix library         : ${optix_LIBRARY}")
message(STATUS "optixu library        : ${optixu_LIBRARY}")
message(STATUS "optix_prime library   : ${optix_prime_LIBRARY}")


find_path(sutil_sdk_INCLUDE_DIR "sutil.h"
    HINTS ${OPTIX_PREFIX}/SDK/sutil
)
find_library(sutil_sdk_LIBRARY "sutil_sdk"
    HINTS ${OPTIX_PREFIX}/SDK-precompiled-samples
)
find_package(GLUT REQUIRED)
find_package(OpenGL REQUIRED)
message(STATUS "sutil_sdk library     : ${sutil_sdk_LIBRARY}")
message(STATUS "sutil_sdk include dir : ${sutil_sdk_INCLUDE_DIR}")



# WHY IS THIS NECESSARY ???
find_package(CUDA REQUIRED)
find_library(CUDA_nvrtc_LIBRARY nvrtc
    PATHS ${CUDA_TOOLKIT_ROOT_DIR}
    PATH_SUFFIXES lib lib64
)


list(APPEND optix_helpers_PUBLIC_HEADERS
    include/optix_helpers/NVRTC_Helper.h
    include/optix_helpers/ProgramManager.h
)
add_library(optix_helpers SHARED
    src/NVRTC_Helper.cpp
    src/ProgramManager.cpp
)
set_target_properties(optix_helpers PROPERTIES 
    PUBLIC_HEADER "${optix_helpers_PUBLIC_HEADERS}"
    # INSTALL_RPATH_USE_LINK_PATH True
)
target_include_directories(optix_helpers
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
        ${OPTIX_INCLUDE_DIR}
        ${sutil_sdk_INCLUDE_DIR}
        ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}
)
target_link_libraries(optix_helpers PUBLIC
    ${optix_LIBRARY}
    ${optixu_LIBRARY}
    ${optix_prime_LIBRARY}
    ${sutil_sdk_LIBRARY}
    ${CUDA_LIBRARIES}
    ${CUDA_nvrtc_LIBRARY}
    GLUT::GLUT
    OpenGL::GL
)
set(NVRTC_INCLUDE_DIRS "${OPTIX_INCLUDE_DIR}")
set(NVRTC_INCLUDE_DIRS "${NVRTC_INCLUDE_DIRS}\\\;${OPTIX_INCLUDE_DIR}/optixu")
set(NVRTC_INCLUDE_DIRS "${NVRTC_INCLUDE_DIRS}\\\;${OPTIX_PREFIX}/SDK/support/mdl-sdk/include")
set(NVRTC_INCLUDE_DIRS "${NVRTC_INCLUDE_DIRS}\\\;${OPTIX_PREFIX}/SDK/sutils")
set(NVRTC_INCLUDE_DIRS "${NVRTC_INCLUDE_DIRS}\\\;${OPTIX_PREFIX}/SDK/cuda")
set(NVRTC_INCLUDE_DIRS "${NVRTC_INCLUDE_DIRS}\\\;${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}")
target_compile_definitions(optix_helpers PUBLIC
    NVRTC_INCLUDE_DIRS="\\\"${NVRTC_INCLUDE_DIRS}\\\""
)

# Tests
add_subdirectory(tests)


include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/installation.cmake)
install_target(optix_helpers)




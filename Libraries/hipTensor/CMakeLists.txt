# MIT License
#
# Copyright (c) 2025 Advanced Micro Devices, Inc. All rights reserved.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.21 FATAL_ERROR)
project(hipTensor_examples LANGUAGES CXX)
include(CTest)

file(RELATIVE_PATH folder_bin ${CMAKE_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/${folder_bin})

if(CMAKE_SYSTEM_NAME MATCHES "Windows")
    message(STATUS "hipTensor examples are only available on Linux")
    return()
else()
    set(ROCM_ROOT
        "/opt/rocm"
        CACHE PATH
        "Root directory of the ROCm installation"
    )
endif()

list(APPEND CMAKE_PREFIX_PATH "${ROCM_ROOT}")

find_package(hiptensor QUIET)
if(NOT hiptensor_FOUND)
    message(
        WARNING
        "hipTensor could not be found, not building hipTensor examples"
    )
    return()
endif()

# Filter to hipTensor-supported architectures (consult hipTensor; 1d19858-style filter)
if(COMMAND hiptensor_is_supported_architecture)
    set(FILTERED_HIP_ARCHITECTURES)
    set(REMOVED_ARCHITECTURES)
    foreach(ARCH ${CMAKE_HIP_ARCHITECTURES})
        hiptensor_is_supported_architecture(${ARCH} IS_SUPPORTED)
        if(IS_SUPPORTED)
            list(APPEND FILTERED_HIP_ARCHITECTURES ${ARCH})
        else()
            list(APPEND REMOVED_ARCHITECTURES ${ARCH})
        endif()
    endforeach()
    if(REMOVED_ARCHITECTURES)
        message(WARNING "hipTensor examples: Removing unsupported architectures: ${REMOVED_ARCHITECTURES}")
    endif()
    if(NOT FILTERED_HIP_ARCHITECTURES)
        message(STATUS "hipTensor examples: No supported architectures found. Not building hipTensor examples")
        return()
    endif()
    set(CMAKE_HIP_ARCHITECTURES ${FILTERED_HIP_ARCHITECTURES})
else()
    # Fallback when hipTensor does not provide the function (e.g. older ROCm)
    include("${CMAKE_CURRENT_LIST_DIR}/../../Common/FilterHIPArchitectures.cmake")
    filter_hip_architectures("hipTensor examples" "gfx908;gfx90a;gfx942;gfx950" SHOULD_SKIP)
    if(SHOULD_SKIP)
        return()
    endif()
endif()

add_subdirectory(contraction)
add_subdirectory(elementwise)
add_subdirectory(reduction)

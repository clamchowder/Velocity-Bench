 # Modifications Copyright (C) 2023 Intel Corporation
 #
 # This Program is subject to the terms of The Unlicense.​
 # If a copy of the license was not distributed with this file, ​
 # you can obtain one at https://spdx.org/licenses/Unlicense.html​
 #​
 #
 # SPDX-License-Identifier: Unlicense
 #

cmake_minimum_required(VERSION 3.10)

project(hashtable_sycl LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(GPU_AOT                 "Build AOT for Intel GPU"      OFF)
option(USE_NVIDIA_BACKEND      "Build for NVIDIA backend"     OFF)
option(USE_AMDHIP_BACKEND      "Build for AMD HIP backend"    OFF)
option(USE_SM                  "Build for specific SM"        OFF)

set(INTEL_GPU_CXX_FLAGS  " -O2 -fsycl -Wall -Wextra -Wno-unused-parameter ")
set(NVIDIA_GPU_CXX_FLAGS " -O3 -fsycl -Wall -Wextra -Wno-unused-parameter ")
set(AMD_GPU_CXX_FLAGS    " -O3 -fsycl -Wall -Wextra -Wno-unused-parameter ")

set(USE_DEFAULT_FLAGS ON)
if("${CMAKE_CXX_FLAGS}" STREQUAL "")
    message(STATUS "Using DEFAULT compilation flags")
else()
    message(STATUS "Overriding DEFAULT compilation flags")
    set(USE_DEFAULT_FLAGS OFF)
endif()

# AOT compilation
if(GPU_AOT)
    message(STATUS "Enabling INTEL backend")
    if(USE_DEFAULT_FLAGS)
        set(CMAKE_CXX_FLAGS "${INTEL_GPU_CXX_FLAGS}") # Default flags for Intel backend
    endif()
    if( (${GPU_AOT} STREQUAL "pvc") OR (${GPU_AOT} STREQUAL "PVC") )
        message(STATUS "Enabling Intel GPU AOT compilation for ${GPU_AOT}")
        string(APPEND CMAKE_CXX_FLAGS " -fsycl-targets=spir64_gen -Xs \"-device 0x0bd5 -revision_id 0x2f\" ")
    else()
        message(STATUS "Using custom AOT compilation flag ${GPU_AOT}")
        string(APPEND CMAKE_CXX_FLAGS " ${GPU_AOT} ")
    endif()
elseif(USE_NVIDIA_BACKEND)
    message(STATUS "Enabling NVIDIA backend")
    if(USE_DEFAULT_FLAGS)
        set(CMAKE_CXX_FLAGS "${NVIDIA_GPU_CXX_FLAGS}") # Default flags for NV backend
    endif()
    if(USE_SM)
        message("-- Building for SM_${USE_SM} compatibility")
        string(APPEND CMAKE_CXX_FLAGS " -fsycl-targets=nvptx64-nvidia-cuda -Xsycl-target-backend --cuda-gpu-arch=sm_${USE_SM} ")
    else()
        message("-- Building for SM_80 compatibility (DEFAULT)")
        string(APPEND CMAKE_CXX_FLAGS " -fsycl-targets=nvptx64-nvidia-cuda -Xsycl-target-backend --cuda-gpu-arch=sm_80 ")
    endif()
elseif(USE_AMDHIP_BACKEND)
    message(STATUS "Enabling AMD HIP backend for ${USE_AMDHIP_BACKEND} AMD architecture")
    if(USE_DEFAULT_FLAGS)
        set(CMAKE_CXX_FLAGS "${AMD_GPU_CXX_FLAGS}") # Default flags for AMD backend (gfx90a for MI250)
    endif()
    string(APPEND CMAKE_CXX_FLAGS " -fsycl-targets=amdgcn-amd-amdhsa -Xsycl-target-backend --offload-arch=${USE_AMDHIP_BACKEND} ")
endif()

set(SOURCES
    ${CMAKE_SOURCE_DIR}/src/main.cpp
    ${CMAKE_SOURCE_DIR}/src/test.cpp
    ${CMAKE_SOURCE_DIR}/src/linearprobing.cpp
)

include_directories(${CMAKE_SOURCE_DIR}/src)

add_executable(${PROJECT_NAME} ${SOURCES})

target_link_libraries(${PROJECT_NAME} sycl OpenCL stdc++fs)

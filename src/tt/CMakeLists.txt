# SPDX-FileCopyrightText: © 2024 Tenstorrent AI ULC
#
# SPDX-License-Identifier: Apache-2.0
#
# This file incorporates work covered by the following copyright and permission notice:
# SPDX-FileCopyrightText: Copyright 2023 The IREE Authors
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
# https://llvm.org/LICENSE.txt

include(ExternalProject)
add_library(TTPJRTTT
    "client.cc"
)

target_link_libraries(TTPJRTTT PUBLIC
    loguru
    TTPJRTCommon
)

target_include_directories(TTPJRTTT PUBLIC
    ${PROJECT_SOURCE_DIR}/src/common
    ${PROJECT_SOURCE_DIR}/third_party/loguru/src/loguru-install/include/
)

add_library(TTPJRTTTDylib
    SHARED
    "dylib_entry_point.cc"
)
target_include_directories(TTPJRTTTDylib PUBLIC ${PROJECT_SOURCE_DIR}/third_party/pjrt_c_api)
target_link_libraries(TTPJRTTTDylib PUBLIC TTPJRTTT)

target_link_libraries(TTPJRTTTDylib PUBLIC coverage_config)

# # Output to the project wide python binary directory tree.
set_target_properties(TTPJRTTTDylib
    PROPERTIES
    PREFIX ""  # Disable "lib" prefix.
    LIBRARY_OUTPUT_NAME pjrt_plugin_tt
    INSTALL_RPATH "$ORIGIN"
)
install(TARGETS TTPJRTTTDylib DESTINATION lib)

target_link_options(TTPJRTTTDylib PRIVATE "-Wl,--no-undefined")

if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build." FORCE)
    # Provide options for the build type
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# Print the current build type
message(STATUS "Current build type: ${CMAKE_BUILD_TYPE}")
# Set compiler flags based on the build type
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(TTPJRTTTDylib PRIVATE DEBUG)
    target_compile_options(TTPJRTTTDylib PRIVATE -g -O0)
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_definitions(TTPJRTTTDylib PRIVATE NDEBUG)
    target_compile_options(TTPJRTTTDylib PRIVATE -O3)
elseif(CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
    target_compile_definitions(TTPJRTTTDylib PRIVATE NDEBUG)
    target_compile_options(TTPJRTTTDylib PRIVATE -O2 -g)
elseif(CMAKE_BUILD_TYPE STREQUAL "MinSizeRel")
    target_compile_definitions(TTPJRTTTDylib PRIVATE NDEBUG)
    target_compile_options(TTPJRTTTDylib PRIVATE -Os)
endif()

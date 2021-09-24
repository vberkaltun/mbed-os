# Copyright (c) 2020 ARM Limited. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

add_subdirectory(cxxsupport)
add_subdirectory(source)

target_include_directories(mbed-core
    INTERFACE
        include
        include/platform
        include/platform/internal
)


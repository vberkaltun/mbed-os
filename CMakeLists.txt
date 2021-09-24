# Copyright (c) 2020-2021 ARM Limited. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# This is the boilerplate for Mbed OS

cmake_minimum_required(VERSION 3.19.0 FATAL_ERROR)

project(mbed-os)

add_subdirectory(extern)

add_library(mbed-core INTERFACE)
add_library(mbed-os INTERFACE)

target_link_libraries(mbed-os
    INTERFACE
        mbed-core
)

# Validate selected C library type
# The C library type selected has to match the library that the target can support
if(${CMAKE_CROSSCOMPILING})
    if(${MBED_C_LIB} STREQUAL "small")
        if(NOT "small" IN_LIST MBED_TARGET_SUPPORTED_C_LIBS)
            if("std" IN_LIST MBED_TARGET_SUPPORTED_C_LIBS)
                message(WARNING
                    "We noticed that target.c_lib is set to `${MBED_C_LIB}`."
                    " As the ${MBED_TARGET} target does not support a small C library for the ${MBED_TOOLCHAIN} toolchain,"
                    " we are using the standard C library instead."
                )
                set(MBED_C_LIB "std" CACHE STRING "")
            endif()
        endif()
    elseif(NOT ${MBED_C_LIB} IN_LIST MBED_TARGET_SUPPORTED_C_LIBS)
        message(FATAL_ERROR
            "Invalid `target.c_lib` ('${MBED_C_LIB}') for '${MBED_TARGET}' target."
            "\nPossible value(s): ${MBED_TARGET_SUPPORTED_C_LIBS}"
        )
    endif()

    # Validate selected printf library
    set(MBED_PRINTF_LIB_TYPES std minimal-printf)
    if(NOT ${MBED_PRINTF_LIB} IN_LIST MBED_PRINTF_LIB_TYPES)
        message(FATAL_ERROR
            "Invalid printf library type '${MBED_PRINTF_LIB}'. Possible values:\n ${MBED_PRINTF_LIB_TYPES}"
        )
    endif()

    mbed_set_cpu_core_definitions(mbed-core)
    if(${MBED_TOOLCHAIN_FILE_USED})
        mbed_set_profile_options(mbed-core ${MBED_TOOLCHAIN})
        mbed_set_c_lib(mbed-core ${MBED_C_LIB})
        mbed_set_printf_lib(mbed-core ${MBED_PRINTF_LIB})

        target_compile_features(mbed-core
            INTERFACE
                c_std_11
                cxx_std_14
        )

    endif()

    target_compile_definitions(mbed-core
        INTERFACE
            ${MBED_TARGET_DEFINITIONS}
            ${MBED_CONFIG_DEFINITIONS}
    )

    # We need to generate a "response file" to pass to the C preprocessor when we preprocess the linker
    # script, because of path length limitations on Windows. We set the response file and bind the path
    # to a global property here. The MBED_TARGET being built queries this global property when it sets
    # the linker script.
    #
    # We must set this global property before the targets subdirectory is added to the project. This is
    # required because the MBED_TARGET depends on the response file. If the path to the response file
    # is not defined when the target requests it the config definitions will not be passed to CPP.
    #
    # TODO: Remove this and find a more idiomatic way of passing compile definitions to CPP without
    # using response files or global properties.
    mbed_generate_options_for_linker(mbed-core RESPONSE_FILE_PATH)
    set_property(GLOBAL PROPERTY COMPILE_DEFS_RESPONSE_FILE ${RESPONSE_FILE_PATH})

    # Add compile definitions for backward compatibility with the toolchain
    # supported. New source files should instead check for __GNUC__ and __clang__
    # for the GCC_ARM and ARM toolchains respectively.
    target_compile_definitions(mbed-core
        INTERFACE
            TOOLCHAIN_GCC_ARM
            TOOLCHAIN_GCC
    )
endif()

# Include mbed.h and config from generate folder
target_include_directories(mbed-core
    INTERFACE
        ${CMAKE_CURRENT_SOURCE_DIR}
)

# Include targets/ first, because any post-build hook needs to be defined
# before other parts of Mbed OS can add greentea tests which require
# mbed_set_post_build().
add_subdirectory(targets)
add_subdirectory(cmsis)
add_subdirectory(drivers)
add_subdirectory(hal)
add_subdirectory(platform)

# TODO: Remove once all example applications have removed it
function(mbed_configure_app_target target)
endfunction()
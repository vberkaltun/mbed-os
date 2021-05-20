# Copyright (c) 2020 ARM Limited. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

add_subdirectory(TARGET_NUCLEO_L073RB EXCLUDE_FROM_ALL)

if(${MBED_TOOLCHAIN} STREQUAL "GCC_ARM")
    set(STARTUP_FILE TOOLCHAIN_GCC_ARM/startup_stm32l073xx.S)
    set(LINKER_FILE TOOLCHAIN_GCC_ARM/STM32L073XB.ld)
elseif(${MBED_TOOLCHAIN} STREQUAL "ARM")
    set(STARTUP_FILE TOOLCHAIN_ARM/startup_stm32l073xx.S)
    set(LINKER_FILE TOOLCHAIN_ARM/stm32l073xb.sct)
endif()

add_library(mbed-stm32l073xb INTERFACE)

target_sources(mbed-stm32l073xb
    INTERFACE
        ${STARTUP_FILE}
        system_clock.c
)

target_include_directories(mbed-stm32l073xb
    INTERFACE
        .
)

mbed_set_linker_script(mbed-stm32l073xb ${CMAKE_CURRENT_SOURCE_DIR}/${LINKER_FILE})

target_link_libraries(mbed-stm32l073xb INTERFACE mbed-stm32l0)

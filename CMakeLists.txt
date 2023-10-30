# Copyright (c) Borislav Stanimirov
# SPDX-License-Identifier: MIT
#
cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

project(my_app_template
    LANGUAGES C CXX
)

# cpm
include(./get_cpm.cmake)

#######################################
# cmake lib
CPMAddPackage(gh:iboB/icm@1.4.6)

list(APPEND CMAKE_MODULE_PATH
    "${icm_SOURCE_DIR}"
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake"
)

######################
# compiler settings

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_C_STANDARD_REQUIRED ON)

if(MSVC)
    # /Zc:preprocessor - incompatible with Windows.h
    add_compile_options(
        -W4
        -D_CRT_SECURE_NO_WARNINGS -Zc:__cplusplus -permissive-
        -volatile:iso -Zc:throwingNew -Zc:templateScope -utf-8 -DNOMINMAX=1
        -wd4251 -wd4275
    )
else()
    add_compile_options(-Wall -Wextra)
endif()

# sanitizers
option(SAN_THREAD "${PROJECT_NAME}: sanitize thread" OFF)
option(SAN_ADDR "${PROJECT_NAME}: sanitize address" OFF)
option(SAN_UB "${PROJECT_NAME}: sanitize undefined behavior" OFF)
option(SAN_LEAK "${PROJECT_NAME}: sanitize leaks" OFF)
mark_as_advanced(SAN_THREAD SAN_ADDR SAN_UB SAN_LEAK)

if(MSVC)
    if(SAN_ADDR)
        add_compile_options(-fsanitize=address)
    endif()
    if(SAN_THREAD OR SAN_UB OR SAN_LEAK)
        message(WARNING "Unsupported sanitizers requested for msvc. Ignored")
    endif()
else()
    if(SAN_THREAD)
        set(sanitizerFlags -fsanitize=thread -g)
        if(SAN_ADDR OR SAN_UB OR SAN_LEAK)
            message(WARNING "Incompatible sanitizer combination requested. Only 'SAN_THREAD' will be respected")
        endif()
    else()
        if(SAN_ADDR)
            list(APPEND sanitizerFlags -fsanitize=address -pthread)
        endif()
        if(SAN_UB)
            list(APPEND sanitizerFlags -fsanitize=undefined)
        endif()
        if(SAN_LEAK)
            if(APPLE)
                message(WARNING "Unsupported leak sanitizer requested for Apple. Ignored")
            else()
                list(APPEND sanitizerFlags -fsanitize=leak)
            endif()
        endif()
    endif()
    if(sanitizerFlags)
        add_compile_options(${sanitizerFlags})
        add_link_options(${sanitizerFlags})
    endif()
endif()

######################
# misc
set_property(GLOBAL PROPERTY USE_FOLDERS ON) # solution folders
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin) # all executables go to bin
set(CMAKE_LINK_DEPENDS_NO_SHARED ON) # no file deps on linker
enable_testing()

######################
# packages
CPMAddPackage(gh:iboB/splat@1.3.1)
CPMAddPackage(gh:iboB/itlib@1.9.2)

######################
# subdirs
add_subdirectory(scratch)

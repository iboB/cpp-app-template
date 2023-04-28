# Copyright (c) Borislav Stanimirov
# SPDX-License-Identifier: MIT
#
cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

project(my_app_template
    LANGUAGES C CXX
)

option(SAN_THREAD "my_app_template: sanitize thread" OFF)
option(SAN_ADDR "my_app_template: sanitize address" OFF)
option(SAN_UB "my_app_template: sanitize undefined behavior" OFF)
option(SAN_LEAK "my_app_template: sanitize leaks" OFF)
mark_as_advanced(SAN_THREAD SAN_ADDR SAN_UB SAN_LEAK)

# cpm
include(./get_cpm.cmake)

#######################################
# cmake lib
CPMAddPackage(gh:iboB/icm@1.4.4)

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

set(compilerFlags "")
set(linkerFlags "")
set(compilerAndLinkerFlags "")

if(MSVC)
    # /Zc:preprocessor - incompatible with Windows.h
    # /Zc:templateScope - TODO: add when msvc 17.5 is the norm
    set(compilerFlags "/W4 -D_CRT_SECURE_NO_WARNINGS /Zc:__cplusplus /permissive-\
        /volatile:iso /Zc:throwingNew /utf-8 -DNOMINMAX=1\
        /w34100 /w34189 /w34701 /w34702 /w34703 /w34706 /w34714 /w34913\
        /wd4251 /wd4275"
    )
else()
    set(compilerFlags "-Wall -Wextra")
endif()

if(MSVC)
    if(SAN_ADDR)
        set(compilerFlags "${compilerFlags} /fsanitize=address")
    endif()
    if(SAN_THREAD OR SAN_UB OR SAN_LEAK)
        message(WARNING "Unsupported sanitizers requested for msvc. Ignored")
    endif()
else()
    if(SAN_THREAD)
        set(compilerAndLinkerFlags "${compilerAndLinkerFlags} -fsanitize=thread -g")
        if(SAN_ADDR OR SAN_UB OR SAN_LEAK)
            message(WARNING "Incompatible sanitizer combination requested. Only 'SAN_THREAD' will be respected")
        endif()
    else()
        if(SAN_ADDR)
            set(compilerAndLinkerFlags "${compilerAndLinkerFlags} -fsanitize=address -pthread")
        endif()
        if(SAN_UB)
            set(compilerAndLinkerFlags "${compilerAndLinkerFlags} -fsanitize=undefined")
        endif()
        if(SAN_LEAK)
            if(APPLE)
                message(WARNING "Unsupported leak sanitizer requested for Apple. Ignored")
            else()
                set(compilerAndLinkerFlags "${compilerAndLinkerFlags} -fsanitize=leak")
            endif()
        endif()
    endif()
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${compilerFlags} ${compilerAndLinkerFlags}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${compilerFlags} ${compilerAndLinkerFlags}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${linkerFlags} ${compilerAndLinkerFlags}")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${linkerFlags} ${compilerAndLinkerFlags}")

######################
# misc
set_property(GLOBAL PROPERTY USE_FOLDERS ON) # solution folders
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin) # all executables go to bin
set(CMAKE_LINK_DEPENDS_NO_SHARED ON) # no file deps on linker
enable_testing()

######################
# packages
CPMAddPackage(gh:iboB/splat@1.3.1)
CPMAddPackage(gh:iboB/itlib@1.9.1)

######################
# subdirs
add_subdirectory(scratch)

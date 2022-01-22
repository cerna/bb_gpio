# ##############################################################################
# Description:  CMakeLists.txt
#
# This file, 'CMakeLists.txt', implements build system rules for BB-GPIO project
#
# Copyright (C) 2021    Jakub Fišer  <jakub DOT fiser AT eryaf DOT com>
#
# This program is free software; you can redistribute it and/or modify it under
# the terms of the GNU Lesser General Public License as published by the Free
# Software Foundation; either version 2.1 of the License, or (at your option)
# any later version.
#
# This program is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE.  See the GNU Lesser General Public License for more
# details.
#
# You should have received a copy of the GNU Lesser General Public License along
# with this library; if not, write to the Free Software Foundation, Inc., 51
# Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
#
# ##############################################################################
# ~~~

cmake_minimum_required(VERSION 3.22)

set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)
set(CMAKE_EXPORT_LINK_COMMANDS TRUE)
set(CMAKE_DISABLE_SOURCE_CHANGES TRUE)
set(CMAKE_DISABLE_IN_SOURCE_BUILD TRUE)

project(
  BB-GPIO
  VERSION 1.0.0
  DESCRIPTION "GPIO driver for Beaglebone Black SBC"
  HOMEPAGE_URL "https://github.com/PocketNC/bb_gpio"
  LANGUAGES C)

if(CMAKE_TOOLCHAIN_FILE)
  cmake_path(IS_RELATIVE CMAKE_TOOLCHAIN_FILE _toolchain_file_is_relative)
  if(_toolchain_file_is_relative)
    file(REAL_PATH "${CMAKE_TOOLCHAIN_FILE}" _toolchain_path BASE_DIRECTORY
         "${CMAKE_BINARY_DIR}" EXPAND_TILDE)
    if(NOT EXISTS "${_toolchain_path}")
      message(
        FATAL_ERROR "Specified Toolchain file ${_toolchain_path} doesn't exist!"
      )
    endif()
    set(CMAKE_TOOLCHAIN_FILE "${_toolchain_path}")
    message(
      WARNING
        "CMAKE_TOOLCHAIN_FILE set to ${CMAKE_TOOLCHAIN_FILE} to resolve the relative path issue. Please, avoid using relative paths!"
    )
    unset(_toolchain_path)
  endif()
  unset(_toolchain_file_is_relative)
endif()

find_package(
  Machinekit-HAL
  COMPONENTS Managed-Runtime Managed-HAL
  REQUIRED)

include(GNUInstallDirs)
include(CPackComponent)
include(CPack)
include(FindPkgConfig)

set(MACHINEKIT_HAL_PACKAGE_PREFIX_PATH "machinekit/hal")
set(MACHINEKIT_HAL_MANAGED_MODULE_DIRECTORY
    "${CMAKE_INSTALL_LIBDIR}/${MACHINEKIT_HAL_PACKAGE_PREFIX_PATH}/module/managed"
)

set(CMAKE_C_STANDARD 11)

set(CMAKE_C_FLAGS_DEBUG "-g -O0")
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
set(CMAKE_C_FLAGS_RELEASE "-O0")
set(CMAKE_CXX_FLAGS_RELEASE "-O0")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "-g -O0")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-g -O0")

message(STATUS "Value of CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
message(STATUS "Value of CMAKE_C_FLAGS: ${CMAKE_C_FLAGS}")
message(STATUS "Value of CMAKE_EXE_LINKER_FLAGS: ${CMAKE_EXE_LINKER_FLAGS}")

pkg_check_modules(LIBGPIOD "libgpiod" REQUIRED IMPORTED_TARGET)

add_library(bb_gpio MODULE)
target_sources(bb_gpio PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/bb_gpio.c"
                               "${CMAKE_CURRENT_SOURCE_DIR}/bb_gpio.h")

target_link_libraries(
  bb_gpio PRIVATE Machinekit::HAL::managed_hal Machinekit::HAL::managed_runtime
                  PkgConfig::LIBGPIOD)

export_rtapi_symbols(TARGET bb_gpio)

set_target_properties(
  bb_gpio
  PROPERTIES OUTPUT_NAME "bb_gpio"
             PREFIX "mod"
             LIBRARY_OUTPUT_DIRECTORY
             "${MACHINEKIT_HAL_MANAGED_MODULE_DIRECTORY}")

install(TARGETS bb_gpio
        LIBRARY DESTINATION "${MACHINEKIT_HAL_MANAGED_MODULE_DIRECTORY}"
                COMPONENT BB_GPIO_Managed_Module_Drivers)

cpack_add_component(BB_GPIO_Managed_Module_Drivers GROUP BB_GPIO_Managed_Module)

cpack_add_component_group(BB_GPIO_Managed_Module)

# SPDX-License-Identifier: GPL-3.0-only
# Copyright (c) 2023 CERN
#
# Please retain this header in all redistributions and modifications of the code.

include_guard(GLOBAL)
cmake_minimum_required(VERSION 3.25)
project(riscv_dbg NONE)

######################################################################
# Set the IP version (checked in CI for consistency with git tag)
set(RISCV_DBG_VERSION 0.1.0 CACHE INTERNAL "Version of the RISCV Debugger IP")

######################################################################
# Download and initialize the dependencies
include(deps/deps.cmake)
# This avoids the python dependencies to be re-installed (only done once at top-level)
if(PROJECT_IS_TOP_LEVEL)
    set(DEPS_INIT_DONE TRUE CACHE BOOL "Indicates deps have been initialized.")
endif()

######################################################################
# Define the riscv debug pkg IP
add_ip(${PROJECT_NAME}
    VENDOR pulp_platform
    LIBRARY pkg
    VERSION ${RISCV_DBG_VERSION}
)
# Add the RTL sources to the IP
ip_sources(${IP} SYSTEMVERILOG
    ${PROJECT_SOURCE_DIR}/src/dm_pkg.sv
)

######################################################################
# Define the riscv debug IP
add_ip(${PROJECT_NAME}
    VENDOR pulp_platform
    LIBRARY ip
    VERSION ${RISCV_DBG_VERSION}
)
# Add the RTL sources to the IP
ip_sources(${IP} SYSTEMVERILOG
    ${PROJECT_SOURCE_DIR}/debug_rom/debug_rom.sv
    ${PROJECT_SOURCE_DIR}/debug_rom/debug_rom_one_scratch.sv
    ${PROJECT_SOURCE_DIR}/src/dm_csrs.sv
    ${PROJECT_SOURCE_DIR}/src/dm_mem.sv
    ${PROJECT_SOURCE_DIR}/src/dm_top.sv
    ${PROJECT_SOURCE_DIR}/src/dm_obi_top.sv
    ${PROJECT_SOURCE_DIR}/src/dmi_cdc.sv
    ${PROJECT_SOURCE_DIR}/src/dmi_jtag.sv
    ${PROJECT_SOURCE_DIR}/src/dmi_jtag_tap.sv
    ${PROJECT_SOURCE_DIR}/src/dm_sba.sv
)
# Link the IP to its dependencies
ip_link(${IP}
    pulp_platform::pkg::riscv_dbg
    lowrisc::ip::prim_cells
    pulp_platform::ip::common_cells
)

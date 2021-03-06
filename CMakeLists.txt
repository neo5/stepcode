#    C M A K E L I S T S . T X T   F O R   S T E P C O D E
#
# This file is Copyright (c) 2010 United States Government as
# represented by the U.S. Army Research Laboratory.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above
# copyright notice, this list of conditions and the following
# disclaimer in the documentation and/or other materials provided
# with the distribution.
#
# 3. The name of the author may not be used to endorse or promote
# products derived from this software without specific prior written
# permission.
#
# THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS
# OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY
# DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
# GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
# WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
# NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

# *******************************************************************
# ***                 STEPcode's CMakeLists.txt                   ***
# *******************************************************************

# This file contains the top level CMakeLists.txt logic for the
# STEPcode software package.

project(SC)

# SC version
set(SC_VERSION_MAJOR 0)
set(SC_VERSION_MINOR 7-dev)
set(SC_VERSION ${SC_VERSION_MAJOR}.${SC_VERSION_MINOR})

# SC ABI version. SC_ABI_SOVERSION should be incremented
# for each release introducing API incompatibilities
set(SC_ABI_SOVERSION 2)
set(SC_ABI_VERSION ${SC_ABI_SOVERSION}.0.0)

# Minimum required version of CMake
cmake_minimum_required(VERSION 2.8.7)
if(COMMAND CMAKE_POLICY)
  CMAKE_POLICY(SET CMP0003 NEW)
endif(COMMAND CMAKE_POLICY)

# CMake derives much of its functionality from modules, typically
# stored in one directory - let CMake know where to find them.
set(SC_CMAKE_DIR "${SC_SOURCE_DIR}/cmake")
if(NOT IS_SUBBUILD)
  set(CMAKE_MODULE_PATH "${SC_CMAKE_DIR};${CMAKE_MODULE_PATH}")
else(NOT IS_SUBBUILD)
  set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${SC_CMAKE_DIR}")
endif(NOT IS_SUBBUILD)

# testing and compilation options, build output dirs, install dirs, uninstall, package creation, etc
include(${SC_CMAKE_DIR}/SC_Build_opts.cmake)

# SC_ADDEXEC and SC_ADDLIB macros, dllimport/export, etc
include(${SC_CMAKE_DIR}/SC_Targets.cmake)

# Macros related to paths
include(${SC_CMAKE_DIR}/SC_Paths.cmake)

# locale stuff
include(${SC_CMAKE_DIR}/SC_Locale.cmake)

# logic related to regenerating the lexer and parser source code
include(${SC_CMAKE_DIR}/SC_Regenerate.cmake)

if(NOT DEFINED SC_SDAI_ADDITIONAL_EXES_SRCS)
  set(SC_SDAI_ADDITIONAL_EXES_SRCS "" CACHE STRING "Source files for additional executables to be linked with SDAI libs")
endif(NOT DEFINED SC_SDAI_ADDITIONAL_EXES_SRCS)

if(NOT DEFINED SC_BUILD_SCHEMAS)
  list(APPEND CONFIG_END_MESSAGES
    "** CMake variable SC_BUILD_SCHEMAS was not set. Defaults to building ALL schemas, which will take a"
    "     while; see http://stepcode.org/mw/index.php?title=STEPcode_CMake_variables#SC_BUILD_SCHEMAS")
  #this makes SC_BUILD_SCHEMAS show up in cmake-gui
  set(SC_BUILD_SCHEMAS "ALL" CACHE string "Semicolon-separated list of paths to EXPRESS schemas to be built")
endif(NOT DEFINED SC_BUILD_SCHEMAS)

list(APPEND CONFIG_END_MESSAGES
  ".. Don't worry about any messages above about missing headers or failed tests, as long as"
  "    you see 'Configuring done' below. Headers and features vary by compiler."
  ".. Generating step can take a while if you are building several schemas.")

# create config headers sc_cf.h and sc_version_string.h
include(${SC_CMAKE_DIR}/SC_Config_Headers.cmake)


################

if(MSVC)
  add_definitions(-D__MSVC__ -D__WIN32__)
  # Disable warning for preferred usage of secure functions (example strcpy should be strcpy_s, ...)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_WARNINGS)
elseif(BORLAND)
  add_definitions(-D__BORLAND__ -D__WIN32__)
else()
  add_definitions(-pedantic -W -Wall -Wundef -Wfloat-equal -Wshadow -Winline -Wno-long-long)
endif()

include_directories(
  ${SC_SOURCE_DIR}/include
  ${SC_BINARY_DIR}/include
 )

add_subdirectory(src/base)
add_subdirectory(src/express)
add_subdirectory(src/exppp)
add_subdirectory(src/exp2cxx)
add_subdirectory(src/exp2python)
add_subdirectory(src/clstepcore)
add_subdirectory(src/cleditor)
add_subdirectory(src/cldai)
add_subdirectory(src/clutils)
if(NOT WIN32) # don't build cllazyfile on windows until export/import macros are in place
  add_subdirectory(src/cllazyfile)
endif(NOT WIN32)
add_subdirectory(include)
add_subdirectory(data)
if(SC_ENABLE_TESTING)
  add_subdirectory(test)
endif(SC_ENABLE_TESTING)
add_subdirectory(doc)

# 'make core' builds everything that isn't generated. for devs.
add_custom_target(core)
add_dependencies(core stepdai check-express stepeditor exp2cxx)

# CONFIG_END_MESSAGES - list of messages to be printed after everything else is done.
# THIS MUST BE LAST to ensure that they are visible to the user without scrolling.
foreach(_msg ${CONFIG_END_MESSAGES})
  message("${_msg}")
endforeach(_msg ${CONFIG_END_MESSAGES})

# Local Variables:
# tab-width: 8
# mode: cmake
# indent-tabs-mode: t
# End:
# ex: shiftwidth=2 tabstop=8


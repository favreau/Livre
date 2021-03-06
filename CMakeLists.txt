#
# Copyright (c) 2011-2015, EPFL/Blue Brain Project
#                          David Steiner <steiner@ifi.uzh.ch>
#                          Ahmet Bilgili <ahmet.bilgili@epfl.ch>
#                          Daniel Nachbaur <daniel.nachbaur@epfl.ch>
#                          Maxim Makhinya  <maxmah@gmail.com>
#
# This file is part of Livre <https://github.com/BlueBrain/Livre>
#


cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(Livre)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMake
  ${CMAKE_SOURCE_DIR}/CMake/common)

include(GitExternal)

set(VERSION_MAJOR "0")
set(VERSION_MINOR "4")
set(VERSION_PATCH "0")
set(VERSION_ABI 3) # OFF or ABI version

set(LIVRE_namespace livre)
set(LIVRE_INCLUDE_NAME livre/core)
set(LIVRE_DESCRIPTION "Largescale Interactive Volume Rendering Engine")
set(COMMON_PROJECT_DOMAIN ch.epfl.bluebrain)

include(Common)
if(NOT MSVC) # cppnetlib does not build on Windows
  git_subproject(RESTBridge https://github.com/BlueBrain/RESTBridge.git 4f16cfb)
endif()
if(APPLE) # WAR otherwise MacPorts X11 (/opt/local) is preferred
  list(REMOVE_ITEM CMAKE_SYSTEM_LIBRARY_PATH /opt/local/lib)
  list(REMOVE_ITEM CMAKE_SYSTEM_PREFIX_PATH /opt/local)
  link_directories(/opt/X11/lib /usr/X11R6/lib)
  include_directories(SYSTEM /opt/X11/include /usr/X11R6/include)
endif()

if(CMAKE_SYSTEM_NAME MATCHES "Linux")
  git_subproject(Tuvok https://github.com/BlueBrain/Tuvok.git bbp)
endif()

common_package(BBPTestData)
common_package(Boost REQUIRED COMPONENTS filesystem program_options thread
                                         system regex unit_test_framework)
common_package(Collage REQUIRED)
common_package(dash REQUIRED)
common_package(Equalizer REQUIRED)
common_package(FlatBuffers)
common_package(GLEW_MX)
common_package(LibJpegTurbo)
common_package(Lunchbox REQUIRED)
common_package(OpenGL REQUIRED)
common_package(OpenMP)
common_package(PNG REQUIRED)
common_package(Qt5Core REQUIRED)
common_package(Qt5OpenGL REQUIRED)
common_package(Qt5Widgets REQUIRED)
common_package(RESTBridge)
common_package(Threads REQUIRED)
common_package(Tuvok)
common_package(VTune)
common_package(zeq)
common_package(ISC)

if(ZEQ_FOUND AND FlatBuffers_FOUND)
  list(APPEND COMMON_PACKAGE_DEFINES LIVRE_USE_REMOTE_DATASOURCE)
  set(LIVRE_USE_REMOTE_DATASOURCE 1)
endif()

common_package_post()

include(EqGLLibraries)
set(LIVRE_DEPENDENT_LIBRARIES vmmlib Lunchbox dash Equalizer)

add_definitions(-DBOOST_PROGRAM_OPTIONS_DYN_LINK) # Fix for windows and shared boost.
add_subdirectory(livre)
add_subdirectory(apps)
add_subdirectory(tests)

include(CPackConfig)

set(DOXYGEN_MAINPAGE_MD README.md)
set(DOXYGEN_EXTRA_INPUT ${PROJECT_SOURCE_DIR}/README.md)
include(DoxygenRule)

#    Copyright (c) 2010-2017, Delft University of Technology
#    All rigths reserved
#
#    This file is part of the Tudat. Redistribution and use in source and
#    binary forms, with or without modification, are permitted exclusively
#    under the terms of the Modified BSD license. You should have received
#    a copy of the license with this file. If not, please or visit:
#    http://tudat.tudelft.nl/LICENSE.

# Specify minimum CMake version required to compile Tudat.
cmake_minimum_required( VERSION 2.8.10 )

# Specific Tudat project name.
project( TudatBundle )

# Package options
option(USE_CSPICE   "Build CSPICE libary, tudat with SPICE support and example applications." ON)
option(USE_JSONCPP  "Build JSONCPP library and example applications." OFF)
option(USE_NRLMSISE00 "Build NRLMISE-00 library and tudat with NRLMSISE support." ON)
option(USE_SOFA     "Build Sofa library." OFF)
option(USE_PAGMO    "Build PaGMO library." OFF)
option(USE_PYGMO    "Build PaGMO library with Python bindings." OFF)

# Set root-directory for code to current source directory.
set(PROJECTROOT "${CMAKE_CURRENT_SOURCE_DIR}/")
message(STATUS ${PROJECT_SOURCE_DIR})
enable_testing( )

# Set the flag to force to use include_directories macro on Apple too
option(APPLE_INCLUDE_FORCE "Force include_directories on Apple too." ON)

# Let CMake find our custom cmake modules/scritpts
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/external/CMake")

#
# Boost
#
# Set the required boost components (for Tudat)
set(BoostComponents filesystem system thread regex date_time unit_test_framework)
# Add additional required boost componets for other libraries
# if these are used.
if(USE_PAGMO)
  list(APPEND BoostComponents serialization chrono atomic)
endif()
if(USE_PYGMO)
  list(APPEND BoostComponents python python3)
endif()
# Pick a boost version
# set(BoostVersion 1.60.0)
# set(BoostSHA1 7f56ab507d3258610391b47fef6b11635861175a)
# set(BoostVersion 1.57.0)
# set(BoostSHA1 e151557ae47afd1b43dc3fac46f8b04a8fe51c12)
set(BoostVersion 1.53.0)
set(BoostSHA1 e6dd1b62ceed0a51add3dda6f3fc3ce0f636a7f3)
# set(BoostVersion 1.48.0)
# set(BoostSHA1 27aced5086e96c6f7b2b684bda2bd515e115da35)
# Build boost
include(add_boost)

#
# SPICE
#
if(USE_CSPICE)
  add_subdirectory( "${PROJECTROOT}/cspice/")
endif()

#
# JSONCPP
#
# Add the JSONCPP project and build
if(USE_JSONCPP)
  option(JSONCPP_WITH_TESTS "Build JSONCPP tests." OFF)
  option(JSONCPP_WITH_PKGCONFIG_SUPPORT "Bulid JSONCPP PKGCONFIG files." OFF)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/jsoncpp/lib CACHE PATH "Path for building JSONCPP library in.")
  add_subdirectory( "${PROJECTROOT}/jsoncpp/")
  # Unset library output dir to prevent screwing with other packages
  unset(CMAKE_ARCHIVE_OUTPUT_DIRECTORY CACHE)
endif()

#
# NRLMSISE
#
# Add the NRLMSISE project and build
if(USE_NRLMSISE00)
  set(NRLMSISE00_WITH_TESTS OFF CACHE BOOL "Building of NRLMSISE00 tests.")
  add_subdirectory( "${PROJECTROOT}/nrlmsise-00/")
endif()

#
# SOFA
#
if(USE_SOFA)
  add_subdirectory( "${PROJECTROOT}/sofa/src")
endif()

#
# PaGMO
#
# Add the PaGMO project and build
if(USE_PAGMO)
  OPTION(BUILD_PYGMO    "Build PaGMO with Python bindings." ${USE_PYGMO})
  OPTION(ENABLE_TESTS   "Build PaGMO with test set."        OFF)
  OPTION(BUILD_EXAMPLES "Build PaGMO with examples."        OFF)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/pagmo/lib"
    CACHE PATH "Path for building static PaGMO library in.")
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/pagmo/lib"
    CACHE PATH "Path for building dynamic PaGMO library in.")
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/pagmo/bin"
    CACHE PATH "Path for building PaGMO binaries in.")
  add_subdirectory( "${PROJECTROOT}/pagmo/")
  unset(CMAKE_ARCHIVE_OUTPUT_DIRECTORY CACHE)
  unset(CMAKE_LIBRARY_OUTPUT_DIRECTORY CACHE)
  unset(CMAKE_RUNTIME_OUTPUT_DIRECTORY CACHE)
endif()

#
# Tudat and applications
#
set(TUDAT_BUILD_DIR "${PROJECTROOT}/tudat")
add_subdirectory( "${PROJECTROOT}/tudat/Tudat/")
add_subdirectory( "${PROJECTROOT}/tudatExampleApplications/satellitePropagatorExamples/SatellitePropagatorExamples/")
if(USE_CSPICE AND USE_JSONCPP)
  add_subdirectory( "${PROJECTROOT}/tudatExampleApplications/libraryExamples/SpiceAndJSON/")
endif()
if(USE_PAGMO)
  add_subdirectory( "${PROJECTROOT}/tudatExampleApplications/libraryExamples/PaGMOEx/")
endif()
add_subdirectory( "${PROJECTROOT}/tudatExampleApplications/templateApplication/TemplateApplication")

PROJECT(dune-mlmc CXX)

# local environment
set( ENV{PATH} "${CMAKE_CURRENT_SOURCE_DIR}/../local/bin:$ENV{PATH}" )
set( ENV{LD_LIBRARY_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/../local/lib:$ENV{LD_LIBRARY_PATH}" )
set( ENV{PKG_CONFIG_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/../local/lib/pkgconfig:$ENV{PKG_CONFIG_PATH}" )

#set minimum cmake version
cmake_minimum_required(VERSION 3.1)
set( CMAKE_COLOR_MAKEFILE ON)


if(NOT (dune-common_DIR OR dune-common_ROOT OR
      "${CMAKE_PREFIX_PATH}" MATCHES ".*dune-common.*"))
    string(REPLACE  ${CMAKE_PROJECT_NAME} dune-common dune-common_DIR
      ${PROJECT_BINARY_DIR})
endif()

#find dune-common and set the module path
find_package(dune-common REQUIRED)
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/modules"
  ${dune-common_MODULE_PATH})

#include the dune macros
include(DuneMacros)

# start a dune project with information from dune.module
dune_project()


dune_enable_all_packages(
   INCLUDE_DIRS ${dune-mlmc}/dune
   MODULE_LIBRARIES mlmc_common
)

add_subdirectory(dune)
add_subdirectory(cmake/modules)
add_subdirectory(doc)

# setup header so they show up in QtCreator project trees
foreach(_dep ${ALL_DEPENDENCIES})
  FILE( GLOB_RECURSE _tmp_dune "${CMAKE_CURRENT_SOURCE_DIR}/../${_dep}/dune/*.hh" )
  set( DUNE_HEADERS ${DUNE_HEADERS} ${_tmp_dune} )
endforeach(_dep ${ALL_DEPENDENCIES})
FILE( GLOB_RECURSE local_header "${CMAKE_CURRENT_SOURCE_DIR}/src/*.hh" )
FILE( GLOB_RECURSE local_dune "${CMAKE_CURRENT_SOURCE_DIR}/dune/*.hh" )
set( COMMON_HEADER ${local_header} ${local_dune} ${DUNE_HEADERS} )
set_source_files_properties( ${COMMON_HEADER} PROPERTIES HEADER_FILE_ONLY 1 )


dune_library_add_sources(mlmc_common SOURCES dune/mlmc/msfem.cc)
add_executable("dune_mlmc" src/dune_mlmc.cc )


#disable most warnings from dependent modules, except gdt,stuff
list(APPEND module_exceptions "dune-multiscale" "dune-stuff" "dune-xt-common" "dune-xt-la" "dune-xt-grid" "dune-xt-functions" "dune-gdt")
foreach(_mod ${ALL_DEPENDENCIES})
  list (FIND module_exceptions "${_mod}" _index)
  if (_index LESS 0)
    dune_module_to_uppercase(_upper_case "${_mod}")
    if(${_mod}_INCLUDE_DIRS)
      foreach( _idir ${${_mod}_INCLUDE_DIRS} )
        add_definitions("-isystem ${_idir}")
      endforeach( _idir )
    endif(${_mod}_INCLUDE_DIRS)
  endif(_index GREATER -1)
endforeach(_mod DEPENDENCIES)

add_format(${CMAKE_CURRENT_SOURCE_DIR})
add_tidy(${CMAKE_CURRENT_SOURCE_DIR})

# finalize the dune project, e.g. generating config.h etc.
finalize_dune_project(GENERATE_CONFIG_H_CMAKE)


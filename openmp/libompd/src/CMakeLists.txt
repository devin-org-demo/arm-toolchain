#
#//===----------------------------------------------------------------------===//
#//
#// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
#// See https://llvm.org/LICENSE.txt for license information.
#// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
#//
#//===----------------------------------------------------------------------===//
#

project (libompd)
cmake_minimum_required(VERSION 3.20.0)

function(is_static_only_platform result_var)
  if(CMAKE_SYSTEM_NAME STREQUAL "Generic" OR 
     COMPILER_RT_BAREMETAL_BUILD OR 
     LIBUNWIND_IS_BAREMETAL)
    set(${result_var} TRUE PARENT_SCOPE)
  else()
    set(${result_var} FALSE PARENT_SCOPE)
  endif()
endfunction()

is_static_only_platform(IS_STATIC_ONLY)
if(NOT IS_STATIC_ONLY)
  add_library (ompd SHARED TargetValue.cpp omp-debug.cpp omp-state.cpp omp-icv.cpp)

  # libompd must not link against libomp, there is no code dependency.
  add_dependencies(ompd omp) # ensure generated import library is created first
endif()

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(LIBOMPD_LD_STD_FLAGS FALSE CACHE BOOL
  "Use -stdlibc++ instead of -libc++ library for C++ ")

if(${LIBOMPD_LD_STD_FLAGS})
#  Find and replace/add libstdc++ to compile flags     
   STRING( FIND "${CMAKE_CXX_FLAGS}" "-stdlib=libc++" OUT )
   if("${OUT}" STREQUAL "-1" )   
      set (CMAKE_CXX_FLAGS "-stdlib=libstdc++ ${CMAKE_CXX_FLAGS}")
   else()
      STRING( REPLACE "-stdlib=libc++" "-stdlib=libstdc++" CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} )
   endif()         

#  Find and replace/add libstdc++ to loader flags
   STRING( FIND "${CMAKE_SHARED_LINKER_FLAGS}" "-stdlib=libc++" OUT )
   if("${OUT}" STREQUAL "-1" )   
      set (CMAKE_SHARED_LINKER_FLAGS "-stdlib=libstdc++ ${CMAKE_SHARED_LINKER_FLAGS}")         
   else()
      STRING( REPLACE "-stdlib=libc++" "-stdlib=libstdc++" CMAKE_SHARED_LINKER_FLAGS ${CMAKE_SHARED_LINKER_FLAGS} )
   endif()
endif()

include_directories (
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${LIBOMP_INCLUDE_DIR}
        ${LIBOMP_SRC_DIR}
)

if(TARGET ompd)
  INSTALL( TARGETS ompd
          LIBRARY DESTINATION ${OPENMP_INSTALL_LIBDIR}
          ARCHIVE DESTINATION ${OPENMP_INSTALL_LIBDIR}
          RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" )
endif()

#-------------------------------------------------------------------------------
# GraphBLAS/CMakeLists.txt:  cmake script for GraphBLAS
#-------------------------------------------------------------------------------

# SuiteSparse:GraphBLAS, Timothy A. Davis, (c) 2017-2020, All Rights Reserved.
# http://suitesparse.com   See GraphBLAS/Doc/License.txt for license.

# CMakeLists.txt: instructions for cmake to build GraphBLAS.
# An ANSI C11 compiler is required.
#
# To compile the GraphBLAS library and demo programs:
#
#   make
#
# To compile just the library
#
#   make library
#
# If that fails for any reason, make sure your compiler supports ANSI C11.  You
# could try changing your compiler, for example:
#
#   cd build
#   CC=icc cmake ..
#   cd ..
#   make
#
# Or, with other compilers:
#
#   CC=xlc cmake ..
#   CC=gcc cmake ..
#
# To run the demos:
#
#   cd ../Demo
#   ./demo
#
# To install GraphBLAS in /usr/local/lib and /usr/local/include:
#
#   sudo make install
#
# To remove all compiled files and libraries (except installed ones):
#
#   make distclean

#-------------------------------------------------------------------------------
# get the version
#-------------------------------------------------------------------------------

# cmake 3.13 is preferred.
cmake_minimum_required ( VERSION 2.8.12 )

message ( STATUS "CMake version: " ${CMAKE_VERSION} )

if ( CMAKE_VERSION VERSION_GREATER "3.0" )
    cmake_policy ( SET CMP0042 NEW )
    cmake_policy ( SET CMP0048 NEW )
endif ( )

# version of SuiteSparse:GraphBLAS
set ( GraphBLAS_DATE "Feb 20, 2020" )
set ( GraphBLAS_VERSION_MAJOR 3 )
set ( GraphBLAS_VERSION_MINOR 2 )
set ( GraphBLAS_VERSION_SUB   0 )

# GraphBLAS C API Specification version, at graphblas.org
set ( GraphBLAS_API_DATE "May 18, 2018" )
set ( GraphBLAS_API_VERSION_MAJOR 1 )
set ( GraphBLAS_API_VERSION_MINOR 2 )
set ( GraphBLAS_API_VERSION_SUB   0 )

if ( CMAKE_MAJOR_VERSION GREATER 2 )
    project ( graphblas
        VERSION "${GraphBLAS_VERSION_MAJOR}.${GraphBLAS_VERSION_MINOR}.${GraphBLAS_VERSION_SUB}" LANGUAGES C )
else ( )
    project ( graphblas C )
endif ( )

#-------------------------------------------------------------------------------
# determine build type
#-------------------------------------------------------------------------------

include ( GNUInstallDirs )

# Uncomment this line for for development only, not for end-users:
# set ( CMAKE_BUILD_TYPE Debug )

if ( NOT CMAKE_BUILD_TYPE )
    set ( CMAKE_BUILD_TYPE Release )
endif ( )

# select "true" to build both dynamic and static libraries:
# set ( BUILD_GRB_STATIC_LIBRARY true )
# set ( BUILD_GRB_STATIC_LIBRARY false )
# or use cmake with -DBUILD_GRB_STATIC_LIBRARY=1

if ( BUILD_GRB_STATIC_LIBRARY )
    message ( STATUS "Building both static and dynamic GraphBLAS libraries" )
else ( )
    message ( STATUS "Building dynamic GraphBLAS library only" )
endif ( )

# select "true" to enable burble, for GraphBLAS development only
# set ( GB_BURBLE true )
  set ( GB_BURBLE false )

if ( GB_BURBLE )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -DGB_BURBLE=1 " )
endif ( )

#-------------------------------------------------------------------------------
# configure GraphBLAS
#-------------------------------------------------------------------------------

find_program ( M4EXISTS m4 )
if ( NOT M4EXISTS )
    message ( FATAL_ERROR "m4 not found, but is required.  Install via your system package manager, or download at http://www.gnu.org/software/m4/m4.html or http://gnuwin32.sourceforge.net for Windows" )
endif ( )

set ( DEC0 "Config/GraphBLAS.h.in" )
execute_process (
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    COMMAND m4 -P ${DEC0}
    OUTPUT_FILE "Config/GraphBLAS.h.tmp"
)

#-------------------------------------------------------------------------------
# Configure Include/GraphBLAS.h and documentation with version number
#-------------------------------------------------------------------------------

configure_file (
    "Config/GraphBLAS.h.tmp"
    "${PROJECT_SOURCE_DIR}/Include/GraphBLAS.h"
)
configure_file (
    "Config/GraphBLAS_version.tex.in"
    "${PROJECT_SOURCE_DIR}/Doc/GraphBLAS_version.tex"
)
configure_file (
    "Config/GraphBLAS_API_version.tex.in"
    "${PROJECT_SOURCE_DIR}/Doc/GraphBLAS_API_version.tex"
)
configure_file (
    "Config/README.md.in"
    "${PROJECT_SOURCE_DIR}/README.md"
)

#-------------------------------------------------------------------------------
# determine what user threading model to use
#-------------------------------------------------------------------------------

include ( FindOpenMP  )
include ( FindThreads )

# FUTURE: rely on CUDA
# enable_language ( CUDA )
# for nvcc, add -DGBCUDA

#-------------------------------------------------------------------------------
# report status
#-------------------------------------------------------------------------------

message ( STATUS "CMAKE build type:          " ${CMAKE_BUILD_TYPE} )

if ( ${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    message ( STATUS "CMAKE C Flags debug:       " ${CMAKE_C_FLAGS_DEBUG} )
else ( )
    message ( STATUS "CMAKE C Flags release:     " ${CMAKE_C_FLAGS_RELEASE} )
endif ( )

message ( STATUS "CMAKE compiler ID:         " ${CMAKE_C_COMPILER_ID} )
message ( STATUS "CMAKE thread library:      " ${CMAKE_THREAD_LIBS_INIT} )
message ( STATUS "CMAKE have pthreads:       " ${CMAKE_USE_PTHREADS_INIT}  )
message ( STATUS "CMAKE have Win32 pthreads: " ${CMAKE_USE_WIN32_THREADS_INIT} )
message ( STATUS "CMAKE have OpenMP:         " ${OPENMP_FOUND} )

#-------------------------------------------------------------------------------
# find the BLAS
#-------------------------------------------------------------------------------

# FUTURE: CBLAS disabled for now because of a linking issue.  The problem is
# that GraphBLAS gets linked with both libgomp and libiomp, when the MKL BLAS
# is used.  The FindOpenMP includes libgomp and FindBLAS with BLA_VENDOR set to
# Intel10_64ilp includes libiomp.  When this happens, incorrect results are
# obtained from cblas_saxpy and cblas_daxpy.  The Intel MKL can work with
# libgomp, but not when libiomp is also included.

#   if ( CMAKE_VERSION VERSION_GREATER "3.13" )
#       # Look for the parallel 64-bit MKL BLAS by default
#       set ( BLA_VENDOR Intel10_64ilp )
#       include ( FindBLAS )
#       if ( ${BLAS_FOUND} )
#           set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -DMKL_ILP64 " )
#       else ( )
#           message ( STATUS "CBLAS : Intel MKL not found" )
#           # FUTURE: enable other BLAS flavors here
#           # set ( BLA_VENDOR ... )
#           # include ( FindBLAS )
#       endif ( )
#   endif ( )

#   if ( ${BLAS_FOUND} )
#       # BLAS_LINKER_FLAGS: uncached list of required linker flags
#       # (excluding -l and -L).
#       message ( STATUS "CBLAS:                     found" )
#       message ( STATUS "CBLAS linker flags:        " ${BLAS_LINKER_FLAGS} )
#       # BLAS_LIBRARIES: list of libraries to link against (may be empty)
#       message ( STATUS "CBLAS libraries:    " ${BLAS_LIBRARIES} )
#   else ( )
#       message ( STATUS "CBLAS:                     not found" )
#   endif ( )

#-------------------------------------------------------------------------------
# include directories for both graphblas and graphblasdemo libraries
#-------------------------------------------------------------------------------

set ( CMAKE_INCLUDE_CURRENT_DIR ON )

include_directories ( Source/Template Source Source/Generated Source/Generator Include Demo/Include )

#-------------------------------------------------------------------------------
# compiler options
#-------------------------------------------------------------------------------

# check which compiler is being used.  If you need to make
# compiler-specific modifications, here is the place to do it.
if ( "${CMAKE_C_COMPILER_ID}" STREQUAL "GNU")
    # cmake 2.8 workaround: gcc needs to be told to do ANSI C11.
    # cmake 3.0 doesn't have this problem.
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -std=c11 -lm -Wno-pragmas " )
    # check all warnings (uncomment for development only)
#   set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -Wall -Wextra -Wpedantic -Werror " )
    # set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -g" )
    if ( CMAKE_C_COMPILER_VERSION VERSION_LESS 4.9 )
        message ( FATAL_ERROR "gcc version must be at least 4.9" )
    endif ( )
elseif ( "${CMAKE_C_COMPILER_ID}" STREQUAL "Intel" )
    # options for icc: also needs -std=c11
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -diag-disable 10397,15552 " )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -qopt-report=5 -qopt-report-phase=vec" )
    # the -mp1 option is important for predictable floating-point results with
    # the icc compiler.  Without, ((float) 1.)/((float) 0.) produces NaN,
    # instead of the correct result, Inf.
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -std=c11 -mp1" )
    # The -g option is useful for the Intel VTune tool, but it should be
    # removed in production.  Comment this line out if not in use:
    # set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -g" )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -qopt-malloc-options=3" )
    # check all warnings and remarks (uncomment for development only):
#   set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -w3 -Wremarks -Werror " )
    if ( CMAKE_C_COMPILER_VERSION VERSION_LESS 19.0 )
        message ( FATAL_ERROR "icc version must be at least 19.0" )
    endif ( )
elseif ( "${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" )
    # options for clang
    if ( CMAKE_C_COMPILER_VERSION VERSION_LESS 3.3 )
        message ( FATAL_ERROR "clang version must be at least 3.3" )
    endif ( )
elseif ( "${CMAKE_C_COMPILER_ID}" STREQUAL "MSVC" )
    # options for MicroSoft Visual Studio
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -wd\"4244\" -wd\"4146\" -wd\"4018\" -wd\"4996\" -wd\"4047\" -wd\"4554\"")
elseif ( "${CMAKE_C_COMPILER_ID}" STREQUAL "PGI" )
    # options for PGI pgcc compiler.  The compiler has a bug, and the
    # -DPGI_COMPILER_BUG causes GraphBLAS to use a workaround.
    set ( CMAKE_C_FLAGS    "${CMAKE_C_FLAGS} -Mnoopenmp -noswitcherror -c11 -lm -DPGI_COMPILER_BUG" )
    set ( CMAKE_CXX_FLAGS  "${CMAKE_C_FLAGS} -Mnoopenmp -D__GCC_ATOMIC_TEST_AND_SET_TRUEVAL=1 -noswitcherror --c++11 -lm -DPGI_COMPILER_BUG" )
endif ( )

if ( ${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_DEBUG}" )
else ( )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_RELEASE}" )
endif ( )

#-------------------------------------------------------------------------------
# dynamic graphblas library properties
#-------------------------------------------------------------------------------

# Notes from Sebastien Villemot (sebastien@debian.org):
# SOVERSION policy: if a binary compiled against the old version of the shared
# library needs recompiling in order to work with the new version, then a
# SO_VERSION increase # is needed. Otherwise not.  Examples of the changes that
# require a SO_VERSION increase:
#
#   - a public function or static variable is removed
#   - the prototype of a public function changes
#   - the integer value attached to a public #define or enum changes
#   - the fields of a public structure are modified
#
# Examples of changes that do not require a SO_VERSION increase:
#
#   - a new public function or static variable is added
#   - a private function or static variable is removed or modified
#   - changes in the internals of a structure that is opaque to the calling
#       program (i.e. is only a pointer manipulated through public functions of
#       the library)
#   - a public enum is extended (by adding a new item at the end, but without
#       changing the already existing items)

file ( GLOB GRAPHBLAS_SOURCES "Source/*.c" "Source/Generated/*.c" )

add_library ( graphblas SHARED ${GRAPHBLAS_SOURCES} )
SET_TARGET_PROPERTIES ( graphblas PROPERTIES
    VERSION ${GraphBLAS_VERSION_MAJOR}.${GraphBLAS_VERSION_MINOR}.${GraphBLAS_VERSION_SUB}
    SOVERSION ${GraphBLAS_VERSION_MAJOR}
    C_STANDARD_REQUIRED 11
    PUBLIC_HEADER "Include/GraphBLAS.h" )
set_property ( TARGET graphblas PROPERTY C_STANDARD 11 )

#-------------------------------------------------------------------------------
# static graphblas library properties
#-------------------------------------------------------------------------------

if ( BUILD_GRB_STATIC_LIBRARY )
    add_library ( graphblas_static STATIC ${GRAPHBLAS_SOURCES} )
    SET_TARGET_PROPERTIES ( graphblas_static PROPERTIES
       VERSION ${GraphBLAS_VERSION_MAJOR}.${GraphBLAS_VERSION_MINOR}.${GraphBLAS_VERSION_SUB}
       OUTPUT_NAME graphblas
       SOVERSION ${GraphBLAS_VERSION_MAJOR}
       C_STANDARD_REQUIRED 11
       PUBLIC_HEADER "Include/GraphBLAS.h" )
   set_property ( TARGET graphblas_static PROPERTY C_STANDARD 11 )
endif ( )

#-------------------------------------------------------------------------------
# select the threading library
#-------------------------------------------------------------------------------

if ( USER_OPENMP )
    # user insists on OpenMP synchronization inside GraphBLAS
    message ( STATUS "cmake -DUSER_OPENMP=1: insisting on using OpenMP" )
    set ( USE_OPENMP true )
elseif ( USER_POSIX )
    # user insists on POSIX synchronization inside GraphBLAS
    message ( STATUS "cmake -DUSER_POSIX=1: insisting on using POSIX" )
    set ( USE_POSIX true )
elseif ( USER_NONE )
    message ( STATUS "cmake -DUSER_NONE=1: insisting on using no threading" )
    set ( USE_NONE true )
else ( )
    # default: automatic selection
    message ( STATUS "Automatic selection of synchronization method for user threads" )
    if ( OPENMP_FOUND )
        set ( USE_OPENMP true )
    elseif ( CMAKE_USE_PTHREADS_INIT )
        set ( USE_POSIX true )
    endif ( )
endif ( )

#-------------------------------------------------------------------------------
# select the math library (not required for Microsoft Visual Studio)
#-------------------------------------------------------------------------------

if ( "${CMAKE_C_COMPILER_ID}" STREQUAL "MSVC" )
    set ( M_LIB "" )
else ( )
    set ( M_LIB "m" )
endif ( )

#-------------------------------------------------------------------------------
# add the threading library
#-------------------------------------------------------------------------------

if ( USE_OPENMP )
    # use OpenMP for user thread synchronization
    message ( STATUS "Using OpenMP to synchronize user threads" )
    target_link_libraries ( graphblas ${M_LIB} ${OpenMP_C_LIBRARIES} )
    if ( BUILD_GRB_STATIC_LIBRARY )
        target_link_libraries ( graphblas_static ${M_LIB} ${OpenMP_C_LIBRARIES} )
    endif ( )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS} -DUSER_OPENMP_THREADS " )
elseif ( USE_POSIX )
    # use POSIX for user thread synchronization
    message ( STATUS "Using POSIX pthreads to synchronize user threads" )
    target_link_libraries ( graphblas ${M_LIB} )
    if ( BUILD_GRB_STATIC_LIBRARY )
        target_link_libraries ( graphblas_static ${M_LIB} )
    endif ( )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -pthread -DUSER_POSIX_THREADS " )
else ( )
    # use no threading at all
    message ( WARNING "No support for user threads; GraphBLAS will not be thread-safe" )
    target_link_libraries ( graphblas ${M_LIB} )
    if ( BUILD_GRB_STATIC_LIBRARY )
        target_link_libraries ( graphblas_static ${M_LIB} )
    endif ( )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -DUSER_NO_THREADS " )
endif ( )

if ( CMAKE_THREAD_LIBS_INIT )
    target_link_libraries ( graphblas ${CMAKE_THREAD_LIBS_INIT} )
    if ( BUILD_GRB_STATIC_LIBRARY )
        target_link_libraries ( graphblas_static ${CMAKE_THREAD_LIBS_INIT} )
    endif ( )
endif ( )

if ( OPENMP_FOUND )
    # use OpenMP for internal parallelism
    message ( STATUS "Using OpenMP for internal parallelism" )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}" )
    target_link_libraries ( graphblas ${M_LIB} ${OpenMP_C_LIBRARIES} )
    if ( BUILD_GRB_STATIC_LIBRARY )
        target_link_libraries ( graphblas_static ${M_LIB} ${OpenMP_C_LIBRARIES} )
    endif ( )
endif ( )

if ( CMAKE_USE_PTHREADS_INIT )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -DHAVE_PTHREADS " )
endif ( )

if ( CMAKE_USE_WIN32_THREADS_INIT )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -DHAVE_WINDOWS_THREADS " )
endif ( )

if ( BLAS_FOUND )
    # use the dense CBLAS
    message ( STATUS "Using dense CBLAS for faster dense matrix/vector operations" )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -DGB_HAS_CBLAS " )
    target_link_libraries ( graphblas ${BLAS_LIBRARIES} )
endif ( )

#-------------------------------------------------------------------------------
# determine the default matrix format
#-------------------------------------------------------------------------------

if ( BYCOL )
    message ( STATUS "cmake -DBYCOL=1: default matrix format: GxB_BY_COL" )
    set ( CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -DBYCOL  " )
endif ( )

#-------------------------------------------------------------------------------
# print final C flags
#-------------------------------------------------------------------------------

message ( STATUS "CMAKE C flags: " ${CMAKE_C_FLAGS} )

#-------------------------------------------------------------------------------
# Demo library
#-------------------------------------------------------------------------------

file ( GLOB DEMO_SOURCES "Demo/Source/*.c" )
add_library ( graphblasdemo SHARED ${DEMO_SOURCES} )
SET_TARGET_PROPERTIES ( graphblasdemo PROPERTIES
    C_STANDARD_REQUIRED 11 )
set_property ( TARGET graphblasdemo PROPERTY C_STANDARD 11 )
target_link_libraries ( graphblasdemo ${M_LIB} graphblas )

if ( BUILD_GRB_STATIC_LIBRARY )
    add_library ( graphblasdemo_static STATIC ${DEMO_SOURCES} )
    SET_TARGET_PROPERTIES ( graphblasdemo_static PROPERTIES
        C_STANDARD_REQUIRED 11 )
    set_property ( TARGET graphblasdemo_static PROPERTY C_STANDARD 11 )
    target_link_libraries ( graphblasdemo_static  graphblas_static )
endif ( )

#-------------------------------------------------------------------------------
# Demo programs
#-------------------------------------------------------------------------------

add_executable ( pagerank_demo "Demo/Program/pagerank_demo.c" )
add_executable ( bfs_demo      "Demo/Program/bfs_demo.c" )
add_executable ( tri_demo      "Demo/Program/tri_demo.c" )
add_executable ( pthread_demo  "Demo/Program/pthread_demo.c" )
add_executable ( openmp_demo   "Demo/Program/openmp_demo.c" )
add_executable ( mis_demo      "Demo/Program/mis_demo.c" )
add_executable ( complex_demo  "Demo/Program/complex_demo.c" )
add_executable ( kron_demo     "Demo/Program/kron_demo.c" )
add_executable ( simple_demo   "Demo/Program/simple_demo.c" )
add_executable ( wildtype_demo "Demo/Program/wildtype_demo.c" )
add_executable ( reduce_demo   "Demo/Program/reduce_demo.c" )
add_executable ( import_demo   "Demo/Program/import_demo.c" )

# Libraries required for Demo programs
target_link_libraries ( pagerank_demo graphblas graphblasdemo )
target_link_libraries ( bfs_demo      graphblas graphblasdemo )
target_link_libraries ( tri_demo      graphblas graphblasdemo )
target_link_libraries ( pthread_demo  graphblas graphblasdemo )
target_link_libraries ( openmp_demo   graphblas graphblasdemo )
target_link_libraries ( mis_demo      graphblas graphblasdemo )
target_link_libraries ( complex_demo  graphblas graphblasdemo )
target_link_libraries ( kron_demo     graphblas graphblasdemo )
target_link_libraries ( simple_demo   graphblasdemo )
target_link_libraries ( wildtype_demo graphblas )
target_link_libraries ( reduce_demo   graphblas )
target_link_libraries ( import_demo   graphblas graphblasdemo )

#-------------------------------------------------------------------------------
# graphblas installation location
#-------------------------------------------------------------------------------

# note that the demo libraries are not installed

if ( BUILD_GRB_STATIC_LIBRARY )

    # install both the dynamic and static libraries
    install ( TARGETS graphblas  graphblas_static
        LIBRARY       DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        ARCHIVE       DESTINATION ${CMAKE_INSTALL_LIBDIR} )

else ( )

    # only install the dynamic library
    install ( TARGETS graphblas
         LIBRARY       DESTINATION ${CMAKE_INSTALL_LIBDIR}
         PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
         ARCHIVE       DESTINATION ${CMAKE_INSTALL_LIBDIR} )

endif ( )



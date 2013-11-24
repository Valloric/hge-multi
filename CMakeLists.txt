cmake_minimum_required( VERSION 2.8 )
project( hge_everything )

option( USE_INTERNAL_OPENAL "Use internal OpenAL" ON )
option( USE_INTERNAL_ZLIB "Use internal ZLib" ON )

#############################################################################

# Turning on this flag tells cmake to emit a compile_commands.json file.
# This file can be used to load compilation flags into YCM. See here for more
# details: http://clang.llvm.org/docs/JSONCompilationDatabase.html
set( CMAKE_EXPORT_COMPILE_COMMANDS 1 )

#############################################################################

if ( CMAKE_CXX_COMPILER_ID STREQUAL "Clang" )
  # The Travis CI build machines don't have libc++ installed
  if ( NOT DEFINED ENV{TRAVIS} )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++" )
  endif()

  # Ninja will by default prevent Clang from outputting diagnostics in color
  if ( CMAKE_GENERATOR STREQUAL "Ninja" )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fcolor-diagnostics" )
  endif()
endif()

#############################################################################

# For MSVC enable UNICODE and compilation on multiple processors
if ( MSVC )
  add_definitions( /DUNICODE /D_UNICODE /Zc:wchar_t-  )
  set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP" )
else()
  # For Clang/GCC, compile in C++11 mode
  set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x" )
endif()

#############################################################################

if (UNIX OR APPLE)
  if ( USE_INTERNAL_OPENAL )
    add_subdirectory( openal-soft )
  endif()

  if ( USE_INTERNAL_ZLIB )
    add_subdirectory( zlib )
  endif()

  add_subdirectory( cximage )
endif()

add_subdirectory( hge )

cmake_minimum_required(VERSION 2.8 )
project( everything )

option(USE_INTERNAL_OPENAL "Use internal OpenAL" ON)

if ( CMAKE_GENERATOR STREQUAL Xcode )
  set( CMAKE_GENERATOR_IS_XCODE true )
endif()

# Check if platform is 64 bit
if( CMAKE_SIZEOF_VOID_P EQUAL 4 )
  set( 64_BIT_PLATFORM 0 )
else()
  set( 64_BIT_PLATFORM 1 )
endif()

#############################################################################

# Turning on this flag tells cmake to emit a compile_commands.json file.
# This file can be used to load compilation flags into YCM. See here for more
# details: http://clang.llvm.org/docs/JSONCompilationDatabase.html
set( CMAKE_EXPORT_COMPILE_COMMANDS 1 )

#############################################################################

if ( CMAKE_CXX_COMPILER_ID STREQUAL "Clang" )
  set( COMPILER_IS_CLANG true )

  # The Travis CI build machines don't have libc++ installed
  if ( NOT DEFINED ENV{TRAVIS} )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++" )
  endif()
endif()

#############################################################################

# Determining the presence of C++11 support in the compiler
set( CPP11_AVAILABLE false )
if ( CMAKE_COMPILER_IS_GNUCXX )
  execute_process(
    COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
  if ( GCC_VERSION VERSION_GREATER 4.6 OR GCC_VERSION VERSION_EQUAL 4.6 )
    set( CPP11_AVAILABLE true )
  endif()
elseif( COMPILER_IS_CLANG )
  set( CPP11_AVAILABLE true )
  set( CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++11" )
  set( CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++" )
endif()

#############################################################################

# For MSVC enable UNICODE and compilation on multiple processors
if ( MSVC )
  add_definitions( /DUNICODE /D_UNICODE /Zc:wchar_t-  )
	set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP" )
endif()

#############################################################################

if ( CPP11_AVAILABLE )
  message( "Your C++ compiler supports C++11, compiling in that mode." )
	set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x" )
else()
  message(
    "Your C++ compiler does NOT support C++11, compiling in C++03 mode." )
endif()

if (USE_INTERNAL_OPENAL)
  add_subdirectory( openal-soft )
endif()

add_subdirectory( hge )

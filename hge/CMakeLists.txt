project(hge_multi)
cmake_minimum_required(VERSION 2.8)

option(BUILD_TUTORIALS "Build the tutorial samples" OFF)
option(BUILD_TOOLS "Build the library tools" OFF)
option(BUILD_SINGLE_LIB "Build a single library for HGE and Helpers" ON)

if(UNIX OR APPLE)
  # Needed for FindVorbis
  set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

  if (USE_INTERNAL_OPENAL)
    set(OPENAL_LIBRARY openal)
    set(OPENAL_INCLUDE_DIR "${OpenAL_SOURCE_DIR}/include")
  else()
    find_package(OpenAL REQUIRED)
  endif()

  find_package(SDL REQUIRED)
  find_package(Vorbis REQUIRED)
  find_package(OpenGL REQUIRED)

  # TODO: HGE tools should also use new zlib instead of internal copies
  if (USE_INTERNAL_ZLIB)
    set(ZLIB_LIBRARY zlibstatic)
    set(ZLIB_INCLUDE_DIR "${zlib_SOURCE_DIR};${zlib_BINARY_DIR}")
  else()
    find_package(ZLIB REQUIRED)
  endif()

  # CxImage required defines
  add_definitions(-DSUPPORT_CXIMAGE)
  set(CXIMAGE_LIBRARY cximage)
  set(CXIMAGE_INCLUDE_DIR "${cximage_SOURCE_DIR}")

  set( HGE_EXTERNAL_INCLUDES
    ${OPENAL_INCLUDE_DIR}
    ${OPENGL_INCLUDE_DIR}
    ${SDL_INCLUDE_DIR}
    ${VORBIS_INCLUDE_DIR}
    ${CXIMAGE_INCLUDE_DIR}
    ${ZLIB_INCLUDE_DIR}
    CACHE STRING "HGE external include dirs"
  )

  include_directories(
    SYSTEM
    ${HGE_EXTERNAL_INCLUDES} )

  set( HGE_INTERNAL_INCLUDES
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/core
    CACHE STRING "HGE internal include dirs" )

  include_directories(
    ${HGE_INTERNAL_INCLUDES} )

  add_definitions(-D_LINUX)

  # The libraries to link against for HGE, and also for applications using it
  set(HGE_LIBRARIES
    ${OPENAL_LIBRARY}
    ${SDL_LIBRARY}
    ${VORBISFILE_LIBRARY}
    ${ZLIB_LIBRARY}
    ${CXIMAGE_LIBRARY}
  )

  set(USE_SHARED SHARED)
elseif(WIN32)
  # Windows
  add_definitions(-D_WINDOWS -D_WIN32)
  set(USE_INTERNAL_ZLIB ON)

  set(HGE_LIBRARIES
    d3d8
    winmm
  )

  if(MSVC AND NOT (CMAKE_BUILD_TYPE MATCHES Debug))
    set(HGE_LIBRARIES ${HGE_LIBRARIES}
      d3dx8
    )
  else()
    #Sorry MinGW32, you're not currently allowed to make a "release"
    add_definitions(-DDEBUG)
    set(HGE_LIBRARIES ${HGE_LIBRARIES}
      d3dx8d
    )
  endif()

  set(USE_SHARED SHARED)
else()
  message(FATAL_ERROR "Unknown platform")
endif(UNIX OR APPLE)

if(WIN32 OR ${BUILD_SINGLE_LIB})
  set(HGE_LIBRARY hge ${HGE_LIBRARIES})
  set(HGE_HELPER_LIBRARY hge ${HGE_LIBRARIES})
else()
  # This is the HGE library and required libraries to link against
  set(HGE_LIBRARY hge ${HGE_LIBRARIES})
  # And this is the HGE & Helper library and required libraries to link against
  set(HGE_HELPER_LIBRARY hge_helpers ${HGE_LIBRARY})
endif()

# Where HGE is located, this also builds the helper library
add_subdirectory(src/core)

if(BUILD_TUTORIALS)
  add_subdirectory(tutorials)
endif(BUILD_TUTORIALS)

if(BUILD_TOOLS)
  add_subdirectory(tools)
endif(BUILD_TOOLS)

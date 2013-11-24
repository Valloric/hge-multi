cmake_minimum_required( VERSION 2.8 )

project( cximage )

find_package( JPEG REQUIRED )
find_package( PNG REQUIRED )

add_definitions(-D_LINUX)

include_directories(
  SYSTEM
  ${JPEG_INCLUDE_DIR}
  ${PNG_INCLUDE_DIR}
  )

file( GLOB_RECURSE SOURCES *.h *.cpp )

add_library(
  ${PROJECT_NAME}
  ${SOURCES} )

target_link_libraries(
  ${PROJECT_NAME}
  ${JPEG_LIBRARY}
  ${PNG_LIBRARY} )


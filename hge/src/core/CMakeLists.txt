project(hge)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

# Common source
set(HGE_SRCS
	common/demo.cpp
	common/random.cpp
	common/timer.cpp
  # Not part of the zlib library but in contrib
  ${zlib_SOURCE_DIR}/contrib/minizip/ioapi.c
  ${zlib_SOURCE_DIR}/contrib/minizip/unzip.c
)

# Platform-specific files
if(UNIX OR APPLE)
	# Unix/Linux and OS X share these
	include_directories(unix)
	set(HGE_SRCS
		${HGE_SRCS}
		unix/graphics.cpp
		unix/ini.cpp
		unix/input.cpp
		unix/power.cpp
		unix/resource.cpp
		unix/sound_openal.cpp
		unix/system.cpp
	)

	if(APPLE)
		set(HGE_SRCS
      ${HGE_SRCS}
      unix/macosx_support.mm)
	endif()
else(UNIX OR APPLE)
	# Win32, though it's probably broken to build
	include_directories(windows unix)
	set(HGE_SRCS
		${HGE_SRCS}
		windows/graphics.cpp
		windows/ini.cpp
		windows/input.cpp
		windows/power.cpp
		windows/resource.cpp
		windows/sound_libbass.cpp
		windows/system.cpp
	)
endif()

set(HS ${hge_multi_SOURCE_DIR}/src/helpers)
set(HGE_HELPER_SRCS
	${HS}/hgeanim.cpp
	${HS}/hgecolor.cpp
	${HS}/hgedistort.cpp
	${HS}/hgefont.cpp
	${HS}/hgegui.cpp
	${HS}/hgeguictrls.cpp
	${HS}/hgeparticle.cpp
	${HS}/hgepmanager.cpp
	${HS}/hgerect.cpp
	${HS}/hgeresource.cpp
	${HS}/hgesprite.cpp
	${HS}/hgestrings.cpp
	${HS}/hgevector.cpp
	${HS}/parser.cpp
	${HS}/resources.cpp
)

if(WIN32 OR ${BUILD_SINGLE_LIB})
	add_library(hge ${USE_SHARED} ${HGE_SRCS} ${HGE_HELPER_SRCS})
	target_link_libraries(hge ${HGE_LIBRARIES})
	set(HGE_TARGETS hge)
else()
	add_library(hge ${USE_SHARED} ${HGE_SRCS})
	target_link_libraries(hge ${HGE_LIBRARIES})

	add_library(hge_helpers ${USE_SHARED} ${HGE_HELPER_SRCS})
	target_link_libraries(hge_helpers hge ${HGE_LIBRARIES})
	set(HGE_TARGETS hge hge_helpers)
endif()

install(TARGETS ${HGE_TARGETS}
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
	DESTINATION include/hge
	FILES_MATCHING PATTERN "*.h"
)

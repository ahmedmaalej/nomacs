# usage:
# cd nomacs/root/dir
# mkdir build
# cd build
# cmake ../ImageLounge -DCMAKE_INSTALL_PREFIX=../release -DCMAKE_BUILD_TYPE=debug (or release, by default)
#       on windows there should be -G "generator name" required as well...
# make
# make install
#
#
PROJECT(nomacs)

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
if(COMMAND cmake_policy)
      cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)


OPTION (ENABLE_RAW "Compile with raw images support (OpenCV, libraw)" ON)
OPTION (ENABLE_WEBP "Compile with webP support (webP)" ON)
OPTION (ENABLE_TIFF "Compile with multi-layer tiff" ON)
OPTION (DISABLE_QT_DEBUG "Disable Qt Debug Messages" OFF)


set(NOMACS_VERSION 1.6.0)
add_definitions(-DNOMACS_VERSION="${NOMACS_VERSION}")

if (CMAKE_BUILD_TYPE STREQUAL "debug" OR CMAKE_BUILD_TYPE STREQUAL "Debug" OR CMAKE_BUILD_TYPE STREQUAL "DEBUG")
    message(STATUS "A debug build. -DDEBUG is defined")
    add_definitions(-DDEBUG)
elseif (NOT MSVC) # debug and release need qt debug outputs on windows
    message(STATUS "A release build (non-debug). Debugging outputs are silently ignored.")
    add_definitions(-DQT_NO_DEBUG_OUTPUT)
endif ()

set(EXE_NAME ${CMAKE_PROJECT_NAME})

#Set the custom CMake module directory where our include/lib finders are
SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

IF(MSVC)
	file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/libs)
	file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/Debug)
	file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/Release)
	file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)	
ENDIF(MSVC)

#set(BUILD_EXAMPLES OFF CACHE BOOL "Build all examples")

# find the Qt
set(QT_MIN_VERSION "4.7.0")
set(QT_USE_GUI 1)
set(QT_USE_QTNETWORK 1)
find_package(Qt4 REQUIRED QtCore QtGui QtNetwork)
include(${QT_USE_FILE})

find_package(PkgConfig)
#IF (NOT PkgConfig_FOUND AND NOT MSVC)
#	message(FATAL_ERROR "PkgConfig not found")
#ENDIF()


# exiv2
pkg_check_modules(EXIV2  exiv2>=0.20)
message(STATUS "exiv2 version by pkg-config: ${EXIV2_VERSION}")
IF (NOT EXIV2_FOUND AND MSVC)
	if(CMAKE_CL_64)
		SET(EXIV_SEARCH_PATH "../exiv2-0.23/msvc64/exiv2lib/x64/" )
	else()
		SET(EXIV_SEARCH_PATH "../exiv2-0.23/msvc64/exiv2lib/Win32/" )
	endif()
	
	find_path(EXIV2_BUILD_PATH NAMES "ReleaseDLL/exiv2.lib"
										"ReleaseDLL/exiv2.dll"
										"DebugDLL/exiv2d.lib"
										"DebugDLL/exiv2d.dll"
					PATHS ${EXIV_SEARCH_PATH}
					DOC "Path to the exiv2 build directory" NO_DEFAULT_PATH)
					
	find_path(EXIV2_INCLUDE_DIRS "exiv2/exiv2.hpp" PATHS "../exiv2-0.23/msvc64/include" DOC "Path to exiv2/exiv2.hpp" NO_DEFAULT_PATH)
	MARK_AS_ADVANCED(EXIV2_INCLUDE_DIRS)
	
	# find_path(EXPAT_BUILD_PATH NAMES "ReleaseDLL/libexpat.dll" "DebugDLL/libexpat.dll" PATHS "../exiv2-0.23/msvc64/expat/Win32" DOC "Path to the expat build directory" NO_DEFAULT_PATH)
	# MARK_AS_ADVANCED(EXPAT_BUILD_PATH)
	
	# find_path(ZLIB_BUILD_PATH NAMES "ReleaseDLL/zlib1.dll" "DebugDLL/zlib1d.dll" PATHS "../exiv2-0.23/msvc64/zlib/Win32/" DOC "Path to the zlib build directory" NO_DEFAULT_PATH)
	# MARK_AS_ADVANCED(ZLIB_BUILD_PATH)
	
    IF(  EXISTS ${EXIV2_BUILD_PATH}/ReleaseDLL/exiv2.dll AND
		EXISTS ${EXIV2_BUILD_PATH}/DebugDLL/exiv2d.dll AND 
		EXISTS ${EXIV2_BUILD_PATH}/ReleaseDLL/exiv2.lib AND 
		EXISTS ${EXIV2_BUILD_PATH}/DebugDLL/exiv2d.lib)
		IF( EXISTS ${EXIV2_BUILD_PATH}/ReleaseDLL/libexpat.dll AND 
			EXISTS ${EXIV2_BUILD_PATH}/DebugDLL/libexpat.dll)
			IF( EXISTS ${EXIV2_BUILD_PATH}/ReleaseDLL/zlib1.dll AND
				EXISTS ${EXIV2_BUILD_PATH}/DebugDLL/zlib1d.dll )
		
				FILE(COPY ${EXIV2_BUILD_PATH}/ReleaseDLL/exiv2.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Release)
				FILE(COPY ${EXIV2_BUILD_PATH}/ReleaseDLL/exiv2.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
				FILE(COPY ${EXIV2_BUILD_PATH}/ReleaseDLL/exiv2.lib DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/libs)
				FILE(COPY ${EXIV2_BUILD_PATH}/DebugDLL/exiv2d.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Debug)
				FILE(COPY ${EXIV2_BUILD_PATH}/DebugDLL/exiv2d.lib DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/libs)

				FILE(COPY ${EXIV2_BUILD_PATH}/ReleaseDLL/libexpat.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Release)
				FILE(COPY ${EXIV2_BUILD_PATH}/ReleaseDLL/libexpat.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
				FILE(COPY ${EXIV2_BUILD_PATH}/DebugDLL/libexpat.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Debug)

				FILE(COPY ${EXIV2_BUILD_PATH}/ReleaseDLL/zlib1.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Release)
				FILE(COPY ${EXIV2_BUILD_PATH}/ReleaseDLL/zlib1.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
				FILE(COPY ${EXIV2_BUILD_PATH}/DebugDLL/zlib1d.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Debug)
				
				SET(EXIV2_LIBRARIES optimized exiv2.lib debug exiv2d.lib)
				SET(EXIV2_LIBRARY_DIRS "")
				SET(EXIV2_FOUND true)
			ELSE ()
				message(WARNING "zlib build directory not found. Needs EXIV2_BUILD_PATH which contains ReleaseDLL/zlib1.dll and DebugDLL/zlib1d.dll")
			ENDIF()
		ELSE()
			message(WARNING "expat build directory not found. Needs EXIV2_BUILD_PATH which contains ReleaseDLL/libexpat.dll and DebugDLL/libexpat.dll")
		ENDIF()
	ELSE()
		message(WARNING "exiv build directory not found. Needs EXIV2_BUILD_PATH which contains ReleaseDLL/exiv2.dll, ReleaseDLL/exiv2.lib, DebugDLL/exiv2d.dll and DebugDLL/exiv2d.lib")
	ENDIF()	
	
ENDIF(NOT EXIV2_FOUND AND MSVC)

find_file(HAVE_EXIV2_HPP exiv2/exiv2.hpp ${EXIV2_INCLUDE_DIRS})
MARK_AS_ADVANCED(HAVE_EXIV2_HPP)
if (HAVE_EXIV2_HPP)
	message(STATUS "exiv2 contains exiv2.hpp, using it")
	add_definitions(-DHAVE_EXIV2_HPP)
else (HAVE_EXIV2_HPP)
	message(STATUS "exiv2 does not contain exiv2.hpp. Using older include set")
endif (HAVE_EXIV2_HPP)

#webp
SET(WEBP_INCLUDE_DIR "")
SET(WEBP_SOURCE "")
IF(ENABLE_WEBP)
	ADD_DEFINITIONS(-DNDEBUG -DWEBP_USE_THREAD)

	FILE(GLOB WEBP_DEC_SRCS
		RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/libwebp/src/dec/*c
	)
	
	FILE(GLOB WEBP_DEMUX_SRCS
		RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/libwebp/src/demux/*c
	)

	FILE(GLOB WEBP_DSP_SRCS
		RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/libwebp/src/dsp/*c
	)

	FILE(GLOB WEBP_ENC_SRCS
		RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/libwebp/src/enc/*c
	)

	FILE(GLOB WEBP_UTILS_SRCS
		RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/libwebp/src/utils/*c
	)

	FILE(GLOB WEBP_MUX_SRCS
		RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/libwebp/src/mux/*c
	)
	SET(WEBP_SOURCE ${WEBP_DEC_SRCS} ${WEBP_DEMUX_SRCS} ${WEBP_DSP_SRCS} ${WEBP_ENC_SRCS} ${WEBP_UTILS_SRCS} ${WEBP_MUX_SRC})
	SET(WEBP_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libwebp/src)
	add_definitions(-DWITH_WEBP)
ENDIF(ENABLE_WEBP)



IF (ENABLE_RAW)

	# NOTE: libraw is used only when there is OpenCV found and used
	# LibRaw
	pkg_check_modules(LIBRAW  libraw>=0.12.0)
        IF (NOT LIBRAW_FOUND AND NOT MSVC)
                message(FATAL_ERROR "libraw not found. It's mandatory when used with ENABLE_RAW enabled") 
        ENDIF()
	IF (NOT LIBRAW_FOUND AND MSVC)
	
        	find_path(LIBRAW_INCLUDE_DIRS "libraw/libraw.h" PATHS "../LibRaw-0.13.5/" DOC "Path to libraw/libraw.h" NO_DEFAULT_PATH)

			if(CMAKE_CL_64)
				SET(LIBRAW_SEARCH_PATH "../LibRaw-0.13.5/buildfiles/x64")
			else()
				SET(LIBRAW_SEARCH_PATH "../LibRaw-0.13.5/buildfiles/Win32")
			endif()
			
        	find_path(LIBRAW_BUILD_PATH NAMES "release/libraw.lib" "release/libraw.dll" "debug/libraw.lib" "debug/libraw.dll"
        	                            PATHS ${LIBRAW_SEARCH_PATH} DOC "Path to the libraw build directory" NO_DEFAULT_PATH)

	        IF( EXISTS ${LIBRAW_BUILD_PATH}/release/libraw.dll AND
                	EXISTS ${LIBRAW_BUILD_PATH}/debug/libraw.dll AND 
        	        EXISTS ${LIBRAW_BUILD_PATH}/release/libraw.lib AND 
	                EXISTS ${LIBRAW_BUILD_PATH}/debug/libraw.lib)

                	FILE(COPY ${LIBRAW_BUILD_PATH}/release/libraw.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Release)
        	        FILE(COPY ${LIBRAW_BUILD_PATH}/release/libraw.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
	                FILE(COPY ${LIBRAW_BUILD_PATH}/release/libraw.lib DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/libs)
                	FILE(COPY ${LIBRAW_BUILD_PATH}/debug/libraw.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Debug)
        	        FILE(COPY ${LIBRAW_BUILD_PATH}/debug/libraw.lib DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/libs)

	                SET(LIBRAW_LIBRARY_DIRS "")
                	SET(LIBRAW_LIBRARIES optimized libraw.lib debug libraw.lib)

                	SET(LIBRAW_FOUND true)
                	SET(LIBRAW_VERSION 0.13.5)
        	ELSE()
                	message(WARNING "libraw build directory not found. Needs path which contains release/libraw.dll, release/libraw.lib, debug/libraw.dll and debug/libraw.lib")
        	ENDIF()
	ENDIF(NOT LIBRAW_FOUND AND MSVC)
    IF (LIBRAW_VERSION VERSION_GREATER 0.14.0 OR LIBRAW_VERSION VERSION_EQUAL 0.14.0 )
      add_definitions(-DLIBRAW_VERSION_14)
    endif()
	          
	# OpenCV
	SET(OpenCV_LIBS "")
	if (PKG_CONFIG_FOUND) # not sure: pkgconfig is needed for old linux  with old old opencv systems
		pkg_check_modules(OpenCV  opencv>=2.1.0)
		SET(OpenCV_LIBS ${OpenCV_LIBRARIES})
	endif(PKG_CONFIG_FOUND)
	IF (OpenCV_LIBS STREQUAL "") 
		find_package(OpenCV 2.1.0 REQUIRED core imgproc)
	ENDIF()
	IF (OpenCV_VERSION VERSION_LESS 2.4.0 AND OpenCV_FOUND AND MSVC) # OpenCV didn't allow to define packages before version 2.4.0 ... nomacs was linking against all libs even if they were not compiled -> error
		string(REGEX REPLACE "\\." "" OpenCV_SHORT_VERSION ${OpenCV_VERSION})
		SET(OpenCV_LIBS "debug;opencv_imgproc${OpenCV_SHORT_VERSION}d;optimized;opencv_imgproc${OpenCV_SHORT_VERSION};debug;opencv_core${OpenCV_SHORT_VERSION}d;optimized;opencv_core${OpenCV_SHORT_VERSION};")
	ENDIF()
	IF (NOT OpenCV_FOUND)
		message(FATAL_ERROR "OpenCV not found. It's mandatory when used with ENABLE_RAW enabled") 
	ELSE()
		add_definitions(-DWITH_OPENCV)
	ENDIF()
	
	
	if (APPLE)
		# OpenCV has really draconian dependencies on mac (generating bundle greater tha 105MB)
		# so I play bad games here - expecting all will work with manually updated library names
		message(STATUS "")
		message(STATUS "APPLE: some hacks with OpenCV libraries linking will be performed (only limited set of libs will be used)")
		message(STATUS "APPLE:    originals: ${OpenCV_LIBS}")
		set(OpenCV_LIBS "opencv_core;opencv_imgproc")
        	message(STATUS "APPLE:          new: ${OpenCV_LIBS}")
		message(STATUS "")
	endif (APPLE)

	# lanczos interpolation is not integrated in opencv 2.1 - all other things work
	IF(${OpenCV_VERSION} EQUAL "2.1.0")
		add_definitions(-DDISABLE_LANCZOS)
	ENDIF()

ENDIF(ENABLE_RAW)

#multi-layer tiff
IF(ENABLE_TIFF)
		
	# we try to grab the OpenCV's libtiff
	IF (MSVC)
		find_path(TIFF_BUILD_PATH NAMES "Release/libtiff.lib" "Debug/libtiffd.lib" PATHS "${OpenCV_3RDPARTY_LIB_DIR_OPT}/../" DOC "Path to the libtiff build directory" NO_DEFAULT_PATH)
		IF(EXISTS "${TIFF_BUILD_PATH}/Release/libtiff.lib" AND EXISTS "${TIFF_BUILD_PATH}/Debug/libtiffd.lib")
			FILE(COPY ${TIFF_BUILD_PATH}/Debug/libtiffd.lib DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/libs)
			FILE(COPY ${TIFF_BUILD_PATH}/Release/libtiff.lib DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/libs)
		ELSE ()
			message(FATAL_ERROR, "could not locate libtiff liberaries. Needs TIFF_BUILD_PATH which contains /Release/libtiff.lib, /Debug/libtiffd.lib")
		ENDIF()
		
		SET(TIFF_LIBRARIES optimized "libtiff" debug "libtiffd")		
		find_path(TIFF_CONFIG_DIR NAMES "tif_config.h" HINTS "${OpenCV_DIR}/3rdparty/libtiff" )
	
		# @stefan we need here the path to opencv/3rdparty/libtiff ... update 10.07.2013 stefan: currently not possible with the cmake of opencv
		find_path(TIFF_INCLUDE_DIR NAMES "tiffio.h" HINTS "${OpenCV_DIR}/../3rdparty/libtiff" )

		IF(TIFF_LIBRARIES AND EXISTS ${TIFF_CONFIG_DIR} AND EXISTS ${TIFF_INCLUDE_DIR})
			add_definitions(-DWITH_LIBTIFF)
		ELSE()
			message(FATAL_ERROR "libtiff not found. It's mandatory when used with ENABLE_TIFF enabled.")
			message(FATAL_ERROR "opencv dir: ${OpenCV_DIR}")
		ENDIF()

	ELSE()
		find_package(TIFF)
		IF(TIFF_FOUND)
			SET(TIFF_CONFIG_DIR "")
			add_definitions(-DWITH_LIBTIFF)
		ELSE()
			message(FATAL_ERROR "libtiff was not found. It's mandatory when used with ENABLE_TIFF enabled.")
		ENDIF()
		
	ENDIF(MSVC)
	
	
ENDIF(ENABLE_TIFF)

	
include_directories (
	${QT_INCLUDES}
	${EXIV2_INCLUDE_DIRS}
	${LIBRAW_INCLUDE_DIRS}
	${OpenCV_INCLUDE_DIRS}
	${CMAKE_CURRENT_BINARY_DIR}
	${CMAKE_SOURCE_DIR}/src
	${WEBP_INCLUDE_DIR}
	${TIFF_INCLUDE_DIR}
	${TIFF_CONFIG_DIR}
)

# set (NOMACS_SOURCES
        # src/DkError.cpp
        # src/DkImage.cpp
        # src/DkMath.cpp
		# src/DkManipulationWidgets.cpp
        # src/DkConnection.cpp
        # src/DkNetwork.cpp
        # src/DkNoMacs.cpp
		# src/DkPong.cpp
        # src/DkTimer.cpp
        # src/DkUtils.cpp
        # src/DkViewPort.cpp
        # src/DkWidgets.cpp
		# src/DkDialog.cpp
        # src/DkSettings.cpp
        # src/BorderLayout.cpp
		# src/DkTransferToolBar.cpp
        # src/DkMenu.cpp
        # src/main.cpp
# )
if (APPLE)
	SET (NOMACS_SOURCES ${NOMACS_SOURCES} macosx/nomacs.icns)
endif (APPLE)


file(GLOB NOMACS_SOURCES "src/*.cpp")
file(GLOB NOMACS_HEADERS "src/*.h")


file(GLOB NOMACS_MOCS "src/*.h")

file(GLOB LIBQPSD_SOURCES "libqpsd/*.cpp")
file(GLOB LIBQPSD_HEADERS "libqpsd/*.h")
file(GLOB LIBQPSD_MOCS "libqpsd/*.h")


if (APPLE)
    SET (NOMACS_SOURCES ${NOMACS_SOURCES} macosx/nomacs.icns)
endif (APPLE)

set (NOMACS_FORMS
        src/nomacs.ui
)

set (NOMACS_RESOURCES
        src/nomacs.qrc
)

set(NOMACS_TRANSLATIONS
		translations/nomacs_als.ts
		translations/nomacs_az.ts
		translations/nomacs_cs.ts		
		translations/nomacs_de.ts
		translations/nomacs_en.ts
		translations/nomacs_es.ts		
		translations/nomacs_fr.ts
		translations/nomacs_it.ts
		translations/nomacs_pl.ts
		translations/nomacs_pt.ts
		translations/nomacs_ru.ts
		translations/nomacs_sk.ts
		translations/nomacs_sl.ts
		translations/nomacs_sr.ts		
		translations/nomacs_zh.ts		
)

if (MSVC) # some settings for visual studio
	SET(NOMACS_RC src/nomacs.rc) #add resource file when compiling with MSVC 
	SET(VERSION_LIB Version.lib)
	SET(EXIV2_LIBRARY_DIRS ${CMAKE_CURRENT_BINARY_DIR}/libs) #add libs directory to library dirs 
else(MSVC)
	SET(NOMACS_RC "")
	SET(VERSION_LIB "")
ENDIF(MSVC)

QT4_ADD_TRANSLATION(NOMACS_QM ${NOMACS_TRANSLATIONS})
QT4_WRAP_UI(NOMACS_UI ${NOMACS_FORMS})
QT4_WRAP_CPP(NOMACS_MOC_SRC ${NOMACS_MOCS})
QT4_ADD_RESOURCES(NOMACS_RCC ${NOMACS_RESOURCES})

QT4_WRAP_CPP(LIBQPSD_MOC_SRC ${LIBQPSD_MOCS})

#IF (EXIV2_FOUND AND LIBRAW_FOUND AND OpenCV_FOUND OR NOT COMPILEWITHOPENCV)
IF (EXIV2_FOUND)
	
	IF(MSVC)
		if (DISABLE_QT_DEBUG)
			message (STATUS "disabling qt debug messages")
			add_definitions(-DQT_NO_DEBUG_OUTPUT)
		endif()


		set(EXE_NAME ${CMAKE_PROJECT_NAME})
		set(DLL_NAME lib${CMAKE_PROJECT_NAME})
		set(LIB_NAME optimized ${DLL_NAME}.lib debug ${DLL_NAME}d.lib)
		LIST(REMOVE_ITEM NOMACS_SOURCES ${CMAKE_SOURCE_DIR}/src/main.cpp)
		link_directories(${LIBRAW_LIBRARY_DIRS} ${OpenCV_LIBRARY_DIRS} ${EXIV2_LIBRARY_DIRS})
		add_executable(${EXE_NAME} WIN32  MACOSX_BUNDLE src/main.cpp ${NOMACS_MOC_SRC_SU} ${NOMACS_QM} ${NOMACS_TRANSLATIONS} ${NOMACS_RC})
		target_link_libraries(${EXE_NAME} ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY} ${QT_QTMAIN_LIBRARY} ${VERSION_LIB} ${LIB_NAME})
		
		set_target_properties(${EXE_NAME} PROPERTIES COMPILE_FLAGS "-DDK_DLL_IMPORT")
		set_target_properties(${EXE_NAME} PROPERTIES LINK_FLAGS_REALLYRELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:WINDOWS /LARGEADDRESSAWARE")
		set_target_properties(${EXE_NAME} PROPERTIES LINK_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:CONSOLE /LARGEADDRESSAWARE")
		set_target_properties(${EXE_NAME} PROPERTIES LINK_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /SUBSYSTEM:CONSOLE /LARGEADDRESSAWARE")
		set_target_properties(${EXE_NAME} PROPERTIES IMPORTED_IMPLIB "")
				
		add_library(${DLL_NAME} SHARED ${NOMACS_SOURCES} ${NOMACS_UI} ${NOMACS_MOC_SRC} ${NOMACS_RCC} ${NOMACS_HEADERS} ${NOMACS_RC} ${LIBQPSD_SOURCES} ${LIBQPSD_HEADERS} ${LIBQPSD_MOC_SRC} ${WEBP_SOURCE})
		target_link_libraries(${DLL_NAME} ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY} ${QT_QTNETWORK_LIBRARY} ${QT_QTMAIN_LIBRARY} ${EXIV2_LIBRARIES} ${LIBRAW_LIBRARIES} ${OpenCV_LIBS} ${VERSION_LIB} ${TIFF_LIBRARIES})
		add_dependencies(${EXE_NAME} ${DLL_NAME})
		
		
		
		SET(CMAKE_SHARED_LINKER_FLAGS_REALLYRELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:WINDOWS /LARGEADDRESSAWARE") # /subsystem:windows does not work due to a bug in cmake (see http://public.kitware.com/Bug/view.php?id=12566)

		set_target_properties(${DLL_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_CURRENT_BINARY_DIR}/libs)
		set_target_properties(${DLL_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_BINARY_DIR}/libs)
		set_target_properties(${DLL_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY_REALLYRELEASE ${CMAKE_CURRENT_BINARY_DIR}/libs)
		
		
		
		set_target_properties(${DLL_NAME} PROPERTIES COMPILE_FLAGS "-DDK_DLL_EXPORT")
		set_target_properties(${DLL_NAME} PROPERTIES LINK_FLAGS_REALLYRELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:WINDOWS /LARGEADDRESSAWARE")
		set_target_properties(${DLL_NAME} PROPERTIES LINK_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:CONSOLE /LARGEADDRESSAWARE")
		set_target_properties(${DLL_NAME} PROPERTIES LINK_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /SUBSYSTEM:CONSOLE /LARGEADDRESSAWARE")
		set_target_properties(${DLL_NAME} PROPERTIES DEBUG_OUTPUT_NAME ${DLL_NAME}d)
				
	ELSE()
		set(EXE_NAME ${CMAKE_PROJECT_NAME})
		link_directories(${LIBRAW_LIBRARY_DIRS} ${OpenCV_LIBRARY_DIRS} ${EXIV2_LIBRARY_DIRS})
		add_executable(${EXE_NAME} WIN32 MACOSX_BUNDLE ${NOMACS_SOURCES} ${NOMACS_UI} ${NOMACS_MOC_SRC} ${NOMACS_RCC} ${NOMACS_HEADERS} ${NOMACS_RC} ${NOMACS_QM} ${NOMACS_TRANSLATIONS} ${LIBQPSD_SOURCES} ${LIBQPSD_HEADERS} ${LIBQPSD_MOC_SRC} ${WEBP_SOURCE})
		target_link_libraries(${EXE_NAME} ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY} ${QT_QTNETWORK_LIBRARY} ${QT_QTMAIN_LIBRARY} ${EXIV2_LIBRARIES} ${LIBRAW_LIBRARIES} ${OpenCV_LIBS} ${VERSION_LIB} ${TIFF_LIBRARIES})

		IF (CMAKE_SYSTEM_NAME MATCHES "Linux")
			SET_TARGET_PROPERTIES(${EXE_NAME} PROPERTIES LINK_FLAGS -fopenmp)
		ENDIF()
	ENDIF(MSVC)
		
	IF (APPLE)
        	# mac's bundle install
	        SET_TARGET_PROPERTIES(${EXE_NAME} PROPERTIES MACOSX_BUNDLE_INFO_PLIST "${CMAKE_SOURCE_DIR}/macosx/Info.plist.in")
        	SET(MACOSX_BUNDLE_ICON_FILE nomacs.icns)
        	SET(MACOSX_BUNDLE_INFO_STRING "${EXE_NAME} ${NOMACS_VERSION}")
        	SET(MACOSX_BUNDLE_GUI_IDENTIFIER "org.nomacs")
        	SET(MACOSX_BUNDLE_LONG_VERSION_STRING "${NOMACS_VERSION}")
        	SET(MACOSX_BUNDLE_BUNDLE_NAME "${EXE_NAME}")
        	SET(MACOSX_BUNDLE_SHORT_VERSION_STRING "${NOMACS_VERSION}")
        	SET(MACOSX_BUNDLE_BUNDLE_VERSION "${NOMACS_VERSION}")
        	SET(MACOSX_BUNDLE_COPYRIGHT "(c) Nomacs team")
        	SET_SOURCE_FILES_PROPERTIES(${CMAKE_SOURCE_DIR}/macosx/nomacs.icns PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

		INSTALL(TARGETS ${EXE_NAME} BUNDLE DESTINATION ${CMAKE_INSTALL_PREFIX})

		# create a "transportable" bundle - all libs into the bundle: "make bundle" after make install
		configure_file(macosx/bundle.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/bundle.cmake @ONLY)
		add_custom_target(bundle ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/bundle.cmake)

	ELSEIF (MSVC) # copy qt dlls and change settings for different projects

		SET(OpenCV_REQUIRED_MODULES core imgproc)
		FOREACH(opencvlib ${OpenCV_REQUIRED_MODULES})
			FILE(GLOB dllpath ${OpenCV_DIR}/bin/Release/opencv_${opencvlib}*.dll)
			FILE(COPY ${dllpath} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Release)
			FILE(COPY ${dllpath} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
			
			FILE(GLOB dllpath ${OpenCV_DIR}/bin/Debug/opencv_${opencvlib}*d.dll)
			FILE(COPY ${dllpath} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Debug)
		ENDFOREACH(opencvlib)
		
		SET (QTLIBLIST QtCore QtGui QtNetwork)
		FOREACH(qtlib ${QTLIBLIST})
			GET_FILENAME_COMPONENT(QT_DLL_PATH_tmp ${QT_QMAKE_EXECUTABLE} PATH)
			FILE(COPY ${QT_DLL_PATH_tmp}/${qtlib}4.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Release)
			FILE(COPY ${QT_DLL_PATH_tmp}/${qtlib}4.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
			FILE(COPY ${QT_DLL_PATH_tmp}/${qtlib}d4.dll DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/Debug)
		ENDFOREACH(qtlib)
		
		
		# copy msvcp110.dll to Really Release
		if (CMAKE_GENERATOR STREQUAL "Visual Studio 11" OR  CMAKE_GENERATOR STREQUAL "Visual Studio 11 Win64")
			SET(VS_VERSION 11)
		else()
			SET(VS_VERSION 10)
		endif()
		
		if (MSVC11)
			GET_FILENAME_COMPONENT(VS_DIR [HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\VisualStudio\\11.0\\Setup\\VS;ProductDir] REALPATH CACHE)
			SET(VS_VERSION 11)
		elseif (MSVC10)
			GET_FILENAME_COMPONENT(VS_DIR [HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\VisualStudio\\10.0\\Setup\\VS;ProductDir] REALPATH CACHE)
			SET(VS_VERSION 10)
		endif()
		
		if(CMAKE_CL_64)
			set(VC_RUNTIME_DIR "${VS_DIR}/VC/redist/x64/Microsoft.VC${VS_VERSION}0.CRT")
		else()
			set(VC_RUNTIME_DIR "${VS_DIR}/VC/redist/x86/Microsoft.VC${VS_VERSION}0.CRT")
		endif()
		find_file(MSVCP NAMES msvcp${VS_VERSION}0.dll PATHS ${VC_RUNTIME_DIR} NO_DEFAULT_PATH)
		find_file(MSVCR NAMES msvcr${VS_VERSION}0.dll PATHS ${VC_RUNTIME_DIR} NO_DEFAULT_PATH)
		if(MSVCP)
			FILE(COPY ${MSVCP} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
			FILE(COPY ${MSVCR} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ReallyRelease)
		else()
			set(ERROR_MESSAGE "\nCould not find library msvcp110.dll. for really release")
			message(STATUS  "${ERROR_MESSAGE}")
		endif()

		# create projects
		SET(CMAKE_CONFIGURATION_TYPES "Debug;Release;ReallyRelease" CACHE STRING "limited configs" FORCE)
		add_definitions(/Zc:wchar_t-)
		SET(CMAKE_CXX_FLAGS_REALLYRELEASE "-W3 -O2 -DQT_NO_DEBUG_OUTPUT")
		SET(CMAKE_EXE_LINKER_FLAGS_REALLYRELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:WINDOWS /LARGEADDRESSAWARE") # /subsystem:windows does not work due to a bug in cmake (see http://public.kitware.com/Bug/view.php?id=12566)
		
		set_target_properties(${EXE_NAME} PROPERTIES LINK_FLAGS_REALLYRELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:WINDOWS /LARGEADDRESSAWARE")
		set_target_properties(${EXE_NAME} PROPERTIES LINK_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /SUBSYSTEM:CONSOLE /LARGEADDRESSAWARE")
		set_target_properties(${EXE_NAME} PROPERTIES LINK_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /SUBSYSTEM:CONSOLE /LARGEADDRESSAWARE")
		
		SET(_moc ${CMAKE_CURRENT_BINARY_DIR}/GeneratedFiles)
		SOURCE_GROUP("Generated Files" FILES ${NOMACS_MOC_SRC} ${LIBQPSD_MOCS} ${NOMACS_RCC} ${NOMACS_UI} ${NOMACS_RC} ${NOMACS_QM})
		set_source_files_properties(${NOMACS_TRANSLATIONS} PROPERTIES HEADER_FILE_ONLY TRUE)
		SOURCE_GROUP("Translations" FILES ${NOMACS_TRANSLATIONS})
		SOURCE_GROUP("libqpsd" FILES ${LIBQPSD_SOURCES} ${LIBQPSD_HEADERS})
		SOURCE_GROUP("libwebp" FILES ${WEBP_SOURCE})
		
		
	ELSE(UNIX)	
	
		# installation
		#  binary
		install(TARGETS ${EXE_NAME} RUNTIME DESTINATION bin LIBRARY DESTINATION lib${LIB_SUFFIX})
		#  desktop file
		install(FILES nomacs.desktop DESTINATION share/applications)
		#  icon
		install(FILES src/img/nomacs.png DESTINATION share/pixmaps)
		#  translations
		install(FILES ${NOMACS_QM} DESTINATION share/nomacs/translations)
                #  manpage
                install(FILES Readme/nomacs.1 DESTINATION share/man/man1)


		# "make dist" target
		string(TOLOWER ${CMAKE_PROJECT_NAME} CPACK_PACKAGE_NAME)
		SET(CPACK_PACKAGE_VERSION "${NOMACS_VERSION}")
		SET(CPACK_SOURCE_GENERATOR "TBZ2")
		SET(CPACK_SOURCE_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}")
		SET(CPACK_IGNORE_FILES "/CVS/;/\\\\.svn/;/\\\\.git/;\\\\.swp$;\\\\.#;/#;\\\\.tar.gz$;/CMakeFiles/;CMakeCache.txt;refresh-copyright-and-license.pl;build;release;")
		SET(CPACK_SOURCE_IGNORE_FILES ${CPACK_IGNORE_FILES})
		INCLUDE(CPack)
		# simulate autotools' "make dist"
		add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)
	ENDIF()

	
ELSE(!EXIV2_FOUND)
	message(WARNING "At least one library has not been found")
ENDIF()

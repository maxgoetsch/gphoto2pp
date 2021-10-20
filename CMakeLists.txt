cmake_minimum_required (VERSION 3.13.4)
project (GPHOTO2PP VERSION 1.0.0 DESCRIPTION "A C++ wrapper for libgphoto2" LANGUAGES CXX)

if(CMAKE_BUILD_TYPE)
    message("Building ${CMAKE_BUILD_TYPE} build.")
elseif(NOT CMAKE_BUILD_TYPE)
    message(FATAL_ERROR "No build type specified. Use -DCMAKE_BUILD_TYPE=[DEBUG|RELEASE] to specify.")
endif(CMAKE_BUILD_TYPE)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules/)

# finds the current version of libgphoto2 installed
find_package(GPHOTO2 REQUIRED)
if(GPHOTO2_FOUND)
	include_directories(${Gphoto2_INCLUDE_DIRS})
	list(APPEND LIBS ${Gphoto2_LIBRARIES})
endif()

include(GNUInstallDirs)

if(UNIX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++11")
	if(CMAKE_BUILD_TYPE MATCHES "RELEASE")
		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2")
	elseif(CMAKE_BUILD_TYPE MATCHES "DEBUG")
		set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g")
	endif(CMAKE_BUILD_TYPE MATCHES "RELEASE")
else()
  # I have never tried it on any other OS. I suspect apple will work.
  message( FATAL_ERROR "I have never tried this on a non Linux machine, please try at your own risk")
endif()

# base path of where binaries should be built
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

# where the library will be built to
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/libs)

############
# gphoto2pp
############
FILE(GLOB GPHOTO2PP_SOURCE_FILES "src/*.cpp")
FILE(GLOB GPHOTO2PP_HPPHEADER_FILES "include/gphoto2pp/*.hpp")
FILE(GLOB GPHOTO2PP_HEADER_FILES "include/gphoto2pp/*.h")

set(GPHOTO2PP_SOURCES ${GPHOTO2PP_SOURCE_FILES})
set(GPHOTO2PP_HEADERS ${GPHOTO2PP_HPPHEADER_FILES} ${GPHOTO2PP_HEADER_FILES})

add_library( gphoto2pp SHARED ${GPHOTO2PP_SOURCES})
target_link_libraries(gphoto2pp pthread ${Gphoto2_LIBRARIES})

target_include_directories(gphoto2pp PUBLIC "./include")

# sets the shared library version
# http://cmake.3232098.n2.nabble.com/Version-in-name-of-shared-library-td7581530.html
set_target_properties(gphoto2pp PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR})

# create pkg-config files for configuration in non-cmake projects
configure_file(gphoto2pp.pc.in gphoto2pp.pc @ONLY)

# only offer install option if they compiled with release mode
if(CMAKE_BUILD_TYPE MATCHES "RELEASE")
	# adds the 'make install' targets to copy the shared library to /usr/local/lib/ directory
	install( TARGETS gphoto2pp DESTINATION ${CMAKE_INSTALL_LIBDIR} )
	# adds all the relevant headers to /usr/local/include/gphoto2pp when ``make install`` is executed
	install( FILES ${GPHOTO2PP_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/gphoto2pp )

	# install the pkg-config config to system folder
	install(FILES ${CMAKE_BINARY_DIR}/gphoto2pp.pc DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)
endif(CMAKE_BUILD_TYPE MATCHES "RELEASE")

###########
# Examples
###########
add_subdirectory( examples )

#############
# Unit Tests
#############
# for now only add unit tests in dev mode
if(CMAKE_BUILD_TYPE MATCHES "DEBUG")
	enable_testing()
	add_subdirectory( tests )
endif(CMAKE_BUILD_TYPE MATCHES "DEBUG")

###############################
# Docs generation with Doxygen
###############################
if(CMAKE_BUILD_TYPE MATCHES "DEBUG")
	find_package( Doxygen )
	if( DOXYGEN_FOUND )
		add_custom_target (doc ${DOXYGEN_EXECUTABLE} ${CMAKE_SOURCE_DIR}/Doxyfile
			WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
			COMMENT "Generating source code documentation with Doxygen." VERBATIM)
	endif() # DOXYGEN_FOUND
endif(CMAKE_BUILD_TYPE MATCHES "DEBUG")

###################
# uninstall target
###################
if(CMAKE_BUILD_TYPE MATCHES "RELEASE")
	configure_file(
		"${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
		"${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
		IMMEDIATE @ONLY)

	add_custom_target(uninstall
		COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
endif(CMAKE_BUILD_TYPE MATCHES "RELEASE")

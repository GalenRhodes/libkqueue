#
# Copyright (c) 2011 Marius Zwicker <marius@mlba-team.de>
#
# Permission to use, copy, modify, and distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
# OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
#

cmake_minimum_required(VERSION 2.8)
INCLUDE (CheckIncludeFiles)

project(kqueue)

set(kqueue_VERSION_MAJOR "2")
set(kqueue_VERSION_MINOR "0")
set(kqueue_VERSION_PATCH "4")
set(kqueue_VERSION "${kqueue_VERSION_MAJOR}.${kqueue_VERSION_MINOR}.${kqueue_VERSION_PATCH}")


#files
file(GLOB_RECURSE INCL include/*.h)
source_group(includes FILES ${INCL})
set(kqueue_HDRS include/sys/event.h)
set(kqueue_MANPAGES kqueue.2)

if(WIN32)
	file(GLOB SRC 
		src/windows/*.h
		src/windows/*.c
        src/common/*.h
		src/*.c
        src/common/map.c
        src/common/filter.c
        src/common/knote.c
        src/common/kevent.c
        src/common/kqueue.c
	)
	add_definitions(
		-DLIBKQUEUE_EXPORTS
		-D_USRDLL
		-D_WINDLL
	)
    if(MINGW)
        SET(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG}  -march=i486")
        SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -march=i486")
    endif()
	set(LIB ${LIB} Ws2_32)
else()
	file(GLOB SRC 
		src/posix/*.h
		src/posix/platform.c
		src/linux/*.h
		src/linux/platform.c
		src/linux/signal.c
		src/linux/socket.c
		src/linux/timer.c
		src/linux/user.c
		src/linux/vnode.c
		src/linux/write.c
		src/linux/read.c
		src/common/*.h
		src/common/filter.c
		src/common/knote.c
		src/common/map.c
		src/common/kevent.c
		src/common/kqueue.c		
	)
	include_directories(
		src/common
	)
	add_definitions(
		-Wall -Werror
		-D_XOPEN_SOURCE=600
		-std=c99
		-fvisibility=hidden
	)
	set(LIB ${LIB} pthread)
endif()
source_group(src FILES ${SRC})

#includes
include_directories(
	include
)

set(KQUEUE_NAME       "kqueue" CACHE STRING "Name of the library (e.g. \"kqueue\" for \"libkqueue\")")
set(INCLUDE_DIRECTORY "kqueue" CACHE STRING "Subdirectory of the include path to install the headers.")

#library (static or shared)
add_library(kqueue_shared SHARED ${SRC} ${INCL})
target_link_libraries(kqueue_shared ${LIB})
set_target_properties(kqueue_shared PROPERTIES DEBUG_POSTFIX "D" LINKER_LANGUAGE C SOVERSION ${kqueue_VERSION} OUTPUT_NAME "${KQUEUE_NAME}")
set(INSTALL_TARGETS kqueue_shared)

option(STATIC_KQUEUE "Enable to build libkqueue as static lib" OFF)

if(STATIC_KQUEUE)
	message("-- building libkqueue as static lib")
    add_library(kqueue_static STATIC ${SRC} ${INCL})
	target_link_libraries(kqueue_static ${LIB})
	set_target_properties(kqueue_static PROPERTIES DEBUG_POSTFIX "D" POSITION_INDEPENDENT_CODE true OUTPUT_NAME "${KQUEUE_NAME}")
	list(APPEND INSTALL_TARGETS kqueue_static)
endif()

if (NOT CMAKE_INSTALL_LIBDIR)
	set(CMAKE_INSTALL_LIBDIR lib)
endif ()

if (UNIX)
	install(TARGETS ${INSTALL_TARGETS} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
	install(FILES ${INCL} DESTINATION "include/${INCLUDE_DIRECTORY}/sys")
	install(FILES ${kqueue_MANPAGES} DESTINATION "share/man/man2")
	
	set(CPACK_GENERATOR "TGZ DEB" CACHE STRING "Installer types to generate. Sensible options include TGZ, RPM and DEB")
endif ()

set(CPACK_PACKAGE_CONTACT			 	"Not Me <noone@foobar.com>")
set(CPACK_PACKAGE_NAME					"LIBKQUEUE0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY	"A user space implementation of the kqueue(2) kernel event notification mechanism")
set(CPACK_RESOURCE_FILE_LICENSE			"${kqueue_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_DESCRIPTION_FILE		"${kqueue_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_VERSION_MAJOR			"${kqueue_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR			"${kqueue_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH			"${kqueue_VERSION_PATCH}")
set(CPACK_PACKAGE_INSTALL_DIRECTORY     "CMake ${CMake_VERSION_MAJOR}.${CMake_VERSION_MINOR}")

if (UNIX)
	set(CPACK_DEBIAN_PACKAGE_VERSION		"${kqueue_VERSION}")
	set(CPACK_DEBIAN_PACKAGE_DEPENDS		"libc6 (>=2.7)")
	
	set(CPACK_STRIP_FILES true CACHE BOOL "Strip libraries when packaging")
endif ()
include (CPack)

#tests
option(KQUEUE_TESTS "Enable to build tests for libkqueue" OFF)
if(KQUEUE_TESTS)
    message("-- Adding tests for libkqueue")
    add_subdirectory(test)
endif()


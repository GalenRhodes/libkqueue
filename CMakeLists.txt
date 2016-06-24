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
set(KQUEUE_VERSION_MAJOR "2")
set(KQUEUE_VERSION_MINOR "0")
set(KQUEUE_VERSION_PATCH "4")
set(KQUEUE_VERSION "${KQUEUE_VERSION_MAJOR}.${KQUEUE_VERSION_MINOR}.${KQUEUE_VERSION_PATCH}")

include (CPack)
set(CPACK_STRIP_FILES					ON)
set(CPACK_PACKAGE_CONTACT			 	"Not Me <noone@foobar.com>")
set(CPACK_PACKAGE_NAME					"LIBKQUEUE0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY	"A user space implementation of the kqueue(2) kernel event notification mechanism")
set(CPACK_RESOURCE_FILE_LICENSE			"${kqueue_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_DESCRIPTION_FILE		"${kqueue_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_VERSION_MAJOR			"${KQUEUE_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR			"${KQUEUE_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH			"${KQUEUE_VERSION_PATCH}")

set(CPACK_DEBIAN_PACKAGE_VERSION		"${KQUEUE_VERSION}")
set(CPACK_DEBIAN_PACKAGE_DEPENDS		"libc6 (>=2.7)")

#files
file(GLOB_RECURSE INCL include/*.h)
source_group(includes FILES ${INCL})

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
		-fpic
		-D_XOPEN_SOURCE=600
		-std=c99
		-fvisibility=hidden
	)
endif()
source_group(src FILES ${SRC})

#includes
include_directories(
	include
)

#library (static or shared)
option(STATIC_KQUEUE "Enable to build libkqueue as static lib" OFF)
option(SHARED_KQUEUE "Enable to build libkqueue as shared lib" ON)
if(STATIC_KQUEUE)
	message("-- building libkqueue as static lib")
    add_library(kqueue STATIC ${SRC} ${INCL})
endif()
if(SHARED_KQUEUE OR NOT STATIC_KQUEUE)
	message("-- building libkqueue as shared lib")
    add_library(kqueue SHARED ${SRC} ${INCL})
endif()


if(WIN32)
	set(LIB ${LIB} Ws2_32)
else()
	set(LIB ${LIB} pthread)
endif()
target_link_libraries(kqueue ${LIB})
set_target_properties(kqueue PROPERTIES DEBUG_POSTFIX "D")

#tests
option(KQUEUE_TESTS "Enable to build tests for libkqueue" OFF)
if(KQUEUE_TESTS)
    message("-- Adding tests for libkqueue")
    add_subdirectory(test)
endif()

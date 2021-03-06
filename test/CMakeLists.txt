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

#files
if(UNIX)
    set(SRC
        main.c
        kevent.c
        test.c
        proc.c
        read.c
        signal.c
        timer.c
        vnode.c
        user.c
    )
else()
    set(SRC
        common.h
        kevent.c
        main.c
        read.c
        test.c
        timer.c
        user.c
        vnode.c
    )
endif()


#includes
include_directories(
	../include
)

if(UNIX)
    add_definitions(
        -rdynamic
    )
    set(LIBS pthread rt)
else()
    set(LIBS wsock32)
endif()

add_executable(libkqueue-test ${SRC})
target_include_directories(libkqueue-test PRIVATE ${CMAKE_SOURCE_DIR})
if(WIN32)
  target_compile_definitions(libkqueue-test
                             PRIVATE
                               _CRT_SECURE_NO_WARNINGS
                               _CRT_NONSTDC_NO_WARNINGS
                               _WINSOCK_DEPRECATED_NO_WARNINGS)
  if(CMAKE_C_COMPILER_ID MATCHES Clang)
    target_compile_options(libkqueue-test PRIVATE -Wno-unused-variable)
  endif()
endif()
target_link_libraries(libkqueue-test kqueue ${LIBS})
set_target_properties(libkqueue-test PROPERTIES DEBUG_POSTFIX "D")


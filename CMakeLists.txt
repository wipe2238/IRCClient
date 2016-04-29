cmake_minimum_required( VERSION 3.1.2 )

project( ircclient LANGUAGES CXX )

option( IRCCLIENT_EXAMPLE "Compile example application" ON )

set( EXAMPLE_SRC  ${PROJECT_SOURCE_DIR}/example )
set( LIB_SRC      ${PROJECT_SOURCE_DIR}/lib )

function( set_flags target )
	set_property( TARGET ${target} PROPERTY CXX_STANDARD 11 )
	set_property( TARGET ${target} PROPERTY CXx_STANDARD_REQUIRED ON )

	if( NOT CMAKE_BUILD_TYPE STREQUAL "Debug" )
		target_compile_definitions( ${target} PRIVATE -DNDEBUG )
	endif()

	include( CheckCXXCompilerFlag )
	check_cxx_compiler_flag( -Wall   CHECK_COMPILER_WALL )
	check_cxx_compiler_flag( -Werror CHECK_COMPILER_WERROR )
	check_cxx_compiler_flag( -Wextra CHECK_COMPILER_WEXTRA )

	if( CHECK_COMPILER_WALL )
		target_compile_options( ${target} PRIVATE -Wall )
	endif()

	if( CHECK_COMPILER_WERROR )
		target_compile_options( ${target} PRIVATE -Werror )
	endif()

	if( CHECK_COMPILER_WEXTRA )
		target_compile_options( ${target} PRIVATE -Wextra )
	endif()
endfunction()

add_library( ircclient STATIC
	${LIB_SRC}/IRCClient.cpp
	${LIB_SRC}/IRCClient.h
	${LIB_SRC}/IRCHandler.cpp
	${LIB_SRC}/IRCHandler.h
	${LIB_SRC}/IRCSocket.cpp
	${LIB_SRC}/IRCSocket.h
)

set_flags( ircclient )

if( WIN32 )
	target_link_libraries( ircclient wsock32 ws2_32 )
endif()

if( IRCCLIENT_EXAMPLE )
	set( THREADS_PREFER_PTHREAD_FLAG ON )
	find_package( Threads REQUIRED )

	add_executable( ircclient-example
		${EXAMPLE_SRC}/Main.cpp
		${EXAMPLE_SRC}/Thread.cpp
		${EXAMPLE_SRC}/Thread.h
	)

	target_include_directories( ircclient-example PRIVATE ${LIB_SRC} )

	target_link_libraries( ircclient-example
		Threads::Threads
		ircclient
	)

	set_flags( ircclient-example )
endif()

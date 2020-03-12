cmake_minimum_required(VERSION 3.0)

project(win_iconv C)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
	set(ICONV_IS_ROOT_PROJECT TRUE)
endif(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)

if(ICONV_IS_ROOT_PROJECT)
	# when iconv is root - uses default settings
	option(WIN_ICONV_INSTALL "add install target" OFF)

	if (WIN_ICONV_INSTALL)
		set(CPACK_GENERATOR "TBZ2")
		include(CPack)
	endif(WIN_ICONV_INSTALL)

	option(WIN_ICONV_BUILD_TEST "build test executable" OFF)
	option(WIN_ICONV_BUILD_EXECUTABLE "build the win_iconv executable" ON)
	option(WIN_ICONV_USE_DLL "library can use external iconv DLL" ON)

else(ICONV_IS_ROOT_PROJECT)
	# Subproject uses root settings and do not create any executables

	if (BUILD_SHARED_LIBS)
		set(WIN_ICONV_INSTALL ON CACHE BOOL "add install target")
		set(WIN_ICONV_USE_DLL ON CACHE BOOL "library can use external iconv DLL")
	else(BUILD_SHARED_LIBS)
		set(WIN_ICONV_INSTALL OFF CACHE BOOL "add install target")
		set(WIN_ICONV_USE_DLL OFF CACHE BOOL "library can use external iconv DLL")
	endif(BUILD_SHARED_LIBS)

	set(WIN_ICONV_BUILD_TEST OFF CACHE BOOL "build test executable")
	set(WIN_ICONV_BUILD_EXECUTABLE OFF CACHE BOOL "build the win_iconv executable")
endif(ICONV_IS_ROOT_PROJECT)

if(WIN_ICONV_BUILD_TEST)
    enable_testing()
endif(WIN_ICONV_BUILD_TEST)

set(WIN_ICONV_SOURCES
# headers
	include/iconv.h
	include/mlang.h
# sources
	src/win_iconv.c
)

if (BUILD_SHARED_LIBS)
	# shared library needs def file
	list(APPEND WIN_ICONV_SOURCES iconv.def)
endif(BUILD_SHARED_LIBS)

# static/dynamic depends on standard BUILD_SHARED_LIBS flag
add_library(iconv ${WIN_ICONV_SOURCES})

target_include_directories(iconv PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:include>
)

add_library(WinIconv::WinIconv ALIAS iconv)

if(MSVC)
	target_compile_definitions(iconv
		PRIVATE
			_CRT_SECURE_NO_WARNINGS
	)
endif(MSVC)

if (WIN_ICONV_USE_DLL)
	target_compile_definitions(iconv
		PRIVATE 
			USE_LIBICONV_DLL
	)
	if(DEFAULT_LIBICONV_DLL)
		target_compile_definitions(iconv
			PRIVATE
				DEFAULT_LIBICONV_DLL=${DEFAULT_LIBICONV_DLL}
		)
	endif(DEFAULT_LIBICONV_DLL)
endif(WIN_ICONV_USE_DLL)

if(BUILD_SHARED_LIBS)
	target_compile_definitions(iconv
		PRIVATE
			MAKE_DLL
	)
endif(BUILD_SHARED_LIBS)

if (WIN_ICONV_INSTALL)
    install(TARGETS iconv
		RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
	)
	install(FILES include/iconv.h
		DESTINATION include
	)
endif (WIN_ICONV_INSTALL)

if(WIN_ICONV_BUILD_EXECUTABLE)

	add_executable(win_iconv win_iconv.c)

	if (WIN_ICONV_USE_DLL)
		target_compile_definitions(win_iconv
			PRIVATE 
				USE_LIBICONV_DLL
		)
		if(DEFAULT_LIBICONV_DLL)
			target_compile_definitions(win_iconv
				PRIVATE
					DEFAULT_LIBICONV_DLL=${DEFAULT_LIBICONV_DLL}
			)
		endif(DEFAULT_LIBICONV_DLL)
	endif(WIN_ICONV_USE_DLL)
	
	if(MSVC)
		target_compile_definitions(win_iconv
			PRIVATE
				_CRT_SECURE_NO_WARNINGS
		)
	endif(MSVC)
	
	target_compile_definitions(win_iconv 
		PRIVATE
			MAKE_EXE
	)

	if (WIN_ICONV_INSTALL)
		install(TARGETS win_iconv
			RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib
		)
	endif (WIN_ICONV_INSTALL)

endif(WIN_ICONV_BUILD_EXECUTABLE)

if(WIN_ICONV_BUILD_TEST)

    add_executable(win_iconv_test win_iconv_test.c)
    add_test(win_iconv_test win_iconv_test)

	if (WIN_ICONV_USE_DLL)
		target_compile_definitions(win_iconv_test
			PRIVATE 
				USE_LIBICONV_DLL
		)
		if(DEFAULT_LIBICONV_DLL)
			target_compile_definitions(win_iconv_test
				PRIVATE
					DEFAULT_LIBICONV_DLL=${DEFAULT_LIBICONV_DLL}
			)
		endif(DEFAULT_LIBICONV_DLL)
	endif(WIN_ICONV_USE_DLL)
	
	if(MSVC)
		target_compile_definitions(win_iconv_test
			PRIVATE
				_CRT_SECURE_NO_WARNINGS
		)
	endif(MSVC)
	
	target_compile_definitions(win_iconv_test 
		PRIVATE
			MAKE_EXE
	)

endif(WIN_ICONV_BUILD_TEST)

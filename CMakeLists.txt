cmake_minimum_required (VERSION 3.22)

project(PROJECT_NAME_RENAME_ME)

set(lib_name ${PROJECT_NAME}_lib)
set(exe_name main)
set(test_exe_name test_main)

# this will create compile_commands.json for clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(BUILD_TESTS "Build tests." ON)

# TARGETS 

	# main library

		# getting all source file except src/main.cpp
		file(GLOB_RECURSE LIB_SOURCE CONFIGURE_DEPENDS "src/*.cpp" "src/*.cxx" "src/*.cc")
		list(REMOVE_ITEM LIB_SOURCE "${PROJECT_SOURCE_DIR}/src/main.cpp")
		
		# creating library target
		add_library(${lib_name})
		target_sources(${lib_name} PRIVATE ${LIB_SOURCE})
		target_include_directories(${lib_name} PUBLIC ${PROJECT_SOURCE_DIR}/include )
		target_compile_features(${lib_name} PUBLIC cxx_std_20)

		# compile options 
		if(MSVC)
			target_compile_options(${lib_name} PUBLIC /W4)
		elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
			target_compile_options(${lib_name} PUBLIC -Wall -Wextra -Wpedantic -glldb)
		else() # gcc and others
			target_compile_options(${lib_name} PUBLIC -Wall -Wextra -Wpedantic)	
		endif()

	# main executable
		# creating executable target
		add_executable(${exe_name})
		target_sources(${exe_name} PRIVATE ${PROJECT_SOURCE_DIR}/src/main.cpp)
		target_link_libraries(${exe_name} PRIVATE ${lib_name})
	
	# tests
	if (BUILD_TESTS) 

		# Google-test dependency 
		include(FetchContent)
		FetchContent_Declare(
				googletest
				GIT_REPOSITORY https://github.com/google/googletest.git
				GIT_TAG        release-1.11.0
		)
		FetchContent_MakeAvailable(googletest)
		set(BUILD_GMOCK ON CACHE BOOL "" FORCE)
		set(BUILD_GTEST ON CACHE BOOL "" FORCE)

		# test executable target
		file(GLOB_RECURSE TEST_SOURCE CONFIGURE_DEPENDS "test/*.cpp" "test/*.cxx" "test/*.cc")
		add_executable(${test_exe_name} ${TEST_SOURCE})
		target_link_libraries(${test_exe_name} PRIVATE ${lib_name} gtest gmock)
	endif()


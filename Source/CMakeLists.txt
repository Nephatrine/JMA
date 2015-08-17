cmake_minimum_required(VERSION 3.2)
project(JMA)

#
# Project Options
#

set(CMAKE_CXX_STANDARD 11)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build Shared Libraries")

if(MSVC)
	option(USE_MSVC_RUNTIME_LIBRARY_DLL "Use MSVC runtime library DLL" ON)
endif()

#
# Build Epoxy
#

set(EPOXY_BUILD_TESTS OFF CACHE BOOL "Build Epoxy Test Programs")
add_subdirectory(External/Epoxy)
include_directories(External/Epoxy/include ${PROJECT_BINARY_DIR}/External/Epoxy/include)

#
# Build GLFW
#

set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "Build the GLFW example programs")
set(GLFW_BUILD_TESTS OFF CACHE BOOL "Build the GLFW test programs")
set(GLFW_BUILD_DOCS OFF CACHE BOOL "Build the GLFW documentation")
set(GLFW_INSTALL OFF CACHE BOOL "Generate installation target")
add_subdirectory(External/GLFW)
include_directories(External/GLFW/include)

#
# COMPILER FLAGS
#

if (MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    if (NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
        foreach (flag CMAKE_C_FLAGS
                        CMAKE_C_FLAGS_DEBUG
                        CMAKE_C_FLAGS_RELEASE
                        CMAKE_C_FLAGS_MINSIZEREL
                        CMAKE_C_FLAGS_RELWITHDEBINFO)
            if (${flag} MATCHES "/MD")
                string(REGEX REPLACE "/MD" "/MT" ${flag} "${${flag}}")
            endif()
            if (${flag} MATCHES "/MDd")
                string(REGEX REPLACE "/MDd" "/MTd" ${flag} "${${flag}}")
            endif()
        endforeach()
    endif()
else()
    include(CheckCCompilerFlag)

    add_definitions(-Wall -ggdb)

	set(CMAKE_REQUIRED_FLAGS "-fvisibility=hidden")
	check_c_compiler_flag("" _EPOXY_HAS_VISIBLITYHIDDEN)
	if (_EPOXY_HAS_VISIBLITYHIDDEN)
		add_definitions(-fvisibility=hidden)
	endif()

	if (MINGW)
		set(CMAKE_REQUIRED_FLAGS "-Wl,--nxcompat")
		check_c_compiler_flag("" _EPOXY_HAS_DEP)
		if (_EPOXY_HAS_DEP)
			set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--nxcompat ${CMAKE_SHARED_LINKER_FLAGS}")
		endif()
		set(CMAKE_REQUIRED_FLAGS "-Wl,--dynamicbase")
		check_c_compiler_flag("" _EPOXY_HAS_ASLR)
		if (_EPOXY_HAS_ASLR)
			set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--dynamicbase ${CMAKE_SHARED_LINKER_FLAGS}")
		endif()
		set(CMAKE_REQUIRED_FLAGS "-Wl,--high-entropy-va")
		check_c_compiler_flag("" _EPOXY_HAS_64ASLR)
		if (_EPOXY_HAS_64ASLR)
			set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--high-entropy-va ${CMAKE_SHARED_LINKER_FLAGS}")
		endif()
	endif()
endif()

#
# Client Target
#

add_executable(JMA-Client
		Client/JMA-Client.rc
		Client/Main.cpp)
target_link_libraries(JMA-Client glfw epoxy ${GLFW_LIBRARIES})
set_target_properties(JMA-Client PROPERTIES
		POSITION_INDEPENDENT_CODE ON)
if(MSVC)
	set_target_properties(JMA-Client PROPERTIES
			LINK_FLAGS "/ENTRY:mainCRTStartup")
endif()

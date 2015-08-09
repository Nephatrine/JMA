cmake_minimum_required(VERSION 3.2)
project(JMA)
set(CMAKE_CXX_STANDARD 14)

# Version Information
set(JMA_VERSION_MAJOR 0)
set(JMA_VERSION_MINOR 1)
set(JMA_VERSION_PATCH 0)
set(JMA_VERSION_EXTRA 0)
set(JMA_VERSION "${GLFW_VERSION_MAJOR}.${GLFW_VERSION_MINOR}")
set(JMA_VERSION_FULL "${GLFW_VERSION}.${GLFW_VERSION_PATCH}${GLFW_VERSION_EXTRA}")
 
# Option Flags
if(MSVC)
	option(USE_MSVC_RUNTIME_LIBRARY_DLL "Use MSVC runtime library DLL" ON)
endif()

# GLFW Library
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "Build the GLFW example programs")
set(GLFW_BUILD_TESTS OFF CACHE BOOL "Build the GLFW test programs")
set(GLFW_BUILD_DOCS OFF CACHE BOOL "Build the GLFW documentation")
set(GLFW_INSTALL OFF CACHE BOOL "Generate installation target")
add_subdirectory(External/GLFW)
include_directories(External/GLFW/include)

# MSVC Configuration
if(MSVC)
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

endif()

# MINGW/GCC Configuration
if(MINGW)
	include(CheckCCompilerFlag)
    set(CMAKE_REQUIRED_FLAGS "-Wl,--nxcompat")
    check_c_compiler_flag("" _JMA_HAS_DEP)
    if (_JMA_HAS_DEP)
        set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--nxcompat ${CMAKE_SHARED_LINKER_FLAGS}")
    endif()
    set(CMAKE_REQUIRED_FLAGS "-Wl,--dynamicbase")
    check_c_compiler_flag("" _JMA_HAS_ASLR)
    if (_JMA_HAS_ASLR)
        set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--dynamicbase ${CMAKE_SHARED_LINKER_FLAGS}")
    endif()
    set(CMAKE_REQUIRED_FLAGS "-Wl,--high-entropy-va")
    check_c_compiler_flag("" _JMA_HAS_64ASLR)
    if (_JMA_HAS_64ASLR)
        set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--high-entropy-va ${CMAKE_SHARED_LINKER_FLAGS}")
    endif()
endif()

# Configure JMA-Version Header
configure_file (
	"${PROJECT_SOURCE_DIR}/JMA-Version.h.in"
	"${PROJECT_BINARY_DIR}/JMA-Version.h"
)
include_directories("${PROJECT_BINARY_DIR}")

# JMA-Client
add_executable(JMA-Client
		Client/JMA-Client.rc
		Client/Main.cpp)
target_link_libraries(JMA-Client glfw ${GLFW_LIBRARIES})
set_target_properties(JMA-Client PROPERTIES
		POSITION_INDEPENDENT_CODE ON)
if(MSVC)
	set_target_properties(JMA-Client PROPERTIES
			LINK_FLAGS "/ENTRY:mainCRTStartup")
endif()

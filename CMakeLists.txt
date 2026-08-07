cmake_minimum_required(VERSION 3.10)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/")

if(NOT DEFINED CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo" FORCE)
endif()

project(hovercraft 
	VERSION 1.00
	LANGUAGES CXX C
	HOMEPAGE_URL "http://www.github.com/michaelzeilfelder/hc1"
	DESCRIPTION "H-Craft Championship")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

if(NOT CMAKE_GENERATOR MATCHES "^Visual Studio")
include(GNUInstallDirs)
endif()

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "${CMAKE_HOST_SYSTEM_NAME} ${CMAKE_HOST_SYSTEM_PROCESSOR} ${CMAKE_CXX_COMPILER_ID}/${CMAKE_CXX_COMPILER_FRONTEND_VARIANT} => ${CMAKE_SYSTEM_NAME} ${CMAKE_SYSTEM_PROCESSOR} ${CMAKE_CXX_COMPILER_ID}/${CMAKE_CXX_COMPILER_FRONTEND_VARIANT}")

if(CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "GNU|AppleClang")
if( NOT CMAKE_OPTIMIZATION_LEVEL )
	set( CMAKE_OPTIMIZATION_LEVEL 3 CACHE STRING "Choose the optimization level of build." FORCE )
endif()

if(NOT CMAKE_CROSSCOMPILING)
	add_compile_options(-march=native)
endif()

if(CMAKE_SYSTEM_PROCESSOR MATCHES "x64|x86_64|AMD64")
	add_compile_options(-mfpmath=sse)
endif()

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
	add_compile_options(-Wno-gnu-alignof-expression)
else()
	add_compile_options(-fexpensive-optimizations)
endif()

	add_compile_options(-fopenmp)
if(NOT CMAKE_CXX_COMPILER_ID MATCHES "AppleClang")
	add_compile_options(-fopenmp-simd)
endif()

add_compile_options(-O${CMAKE_OPTIMIZATION_LEVEL})
add_compile_options(-ftree-vectorize)
add_compile_options(-fno-rtti)
add_compile_options(-fno-exceptions)
add_compile_options(-D_FILE_OFFSET_BITS=64)
add_compile_options(-fdata-sections)
add_compile_options(-fpermissive)
add_compile_options(-ffunction-sections)
add_compile_options(-Wno-array-bounds)
add_compile_options(-Wno-unused-parameter)
add_compile_options(-Wno-unused-result)
add_compile_options(-Wno-pedantic)
add_compile_options(-Wno-narrowing)
add_compile_options(-fno-strict-aliasing)
add_compile_options(-Wall)
add_link_options(-Wl,--gc-sections)

if( CMAKE_BUILD_TYPE MATCHES Debug )
	add_compile_options(-g)
endif()
if(CMAKE_BUILD_TYPE MATCHES ReleaseStrip)
	add_link_options(-Wl,-s)
endif()
elseif(CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "MSVC")
	if( NOT CMAKE_OPTIMIZATION_LEVEL )
		set( CMAKE_OPTIMIZATION_LEVEL 2 CACHE STRING "Choose the optimization level." FORCE )
	endif()
	add_compile_options(/O${CMAKE_OPTIOMIZATION_LEVEL})
	string(REPLACE "/RTC1" "" CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG}")
	string(REPLACE "/RTC1" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
	add_compile_options(/openmp:experimental)
	add_compile_options(/Zc:__cplusplus)
	add_compile_options(/std:c++latest)
	add_compile_options(/Zc:preprocessor)
	add_compile_options(/Z7)
	add_compile_options(/Gy)					# Function-level linking (puts each function in its own section)
	add_compile_options(/Gw)					# Optimize global data (puts each variable in its own section)
	if (NOT CMAKE_CROSSCOMPILING)
		add_compile_definitions(_CRT_SECURE_NO_WARNINGS _CRT_NONSTDC_NO_DEPRECATE _WIN32)
		if (CMAKE_VS_PLATFORM_NAME MATCHES "x64|Win32")
			#add_compile_options(/Gv)
		endif()
	endif()
	if (CMAKE_BUILD_TYPE MATCHES ReleaseStrip)
		add_link_options(/DEBUG:NONE)
		add_link_options(/OPT:REF)				# Eliminate unreferenced functions and data
		add_link_options(/VERBOSE:REF)			# Print out everything the linker throws away
	endif()
	if (CMAKE_BUILD_TYPE MATCHES Debug)
		add_compile_options(/Zi)
		add_link_options(/DEBUG:FULL)
	endif()
	add_link_options(/INCREMENTAL:NO)
else()
	message(FATAL_ERROR "C++ Compiler Frontend Variant not supported!")
endif()

if(UNIX)
find_package(X11 REQUIRED)
find_package(X11 COMPONENTS Xrandr REQUIRED)
cmake_policy(SET CMP0072 NEW)   #OpenGL_GL_PREFERENCE to GLVND
endif()
find_package(OpenGL REQUIRED)
find_package(libzip REQUIRED)
find_package(PNG REQUIRED)
find_package(Freetype REQUIRED)
find_package(ALUT REQUIRED)
find_package(Vorbis REQUIRED)
find_package(SDL REQUIRED)
find_package(Irrlicht REQUIRED)

if(WIN32)
	ENABLE_LANGUAGE(RC)
	set(HOVERCRAFT_SOURCES ${CMAKE_SOURCE_DIR}/src/resources.rc ${HOVERCRAFT_SOURCES})
endif()

file(GLOB_RECURSE HOVERCRAFT_CXX_SOURCES  ${CMAKE_SOURCE_DIR}/src/*.cpp)
file(GLOB_RECURSE HOVERCRAFT_C_SOURCES    ${CMAKE_SOURCE_DIR}/src/*.c  )
set_source_files_properties(${HOVERCRAFT_C_SOURCES} PROPERTIES COMPILE_FLAGS -Wno-no-rtti)

file(GLOB_RECURSE HOVERCRAFT_INCLUDES     ${CMAKE_SOURCE_DIR}/src/*.h  )

add_compile_definitions(
	_IRR_SHARED_LIB_
	$<$<CONFIG:Relese>:NDEBUG=1>
	$<$<CONFIG:Release>:HOVER_RELEASE>
	$<$<CONFIG:ReleaseStrip>:NDEBUG=1>
	$<$<CONFIG:ReleaseStrip>:HOVER_RELEASE>
	$<$<CONFIG:Debug>:DEBUG>
	$<$<CONFIG:Debug>:_DEBUG>
)

add_executable(${PROJECT_NAME} ${HOVERCRAFT_CXX_SOURCES} ${HOVERCRAFT_C_SOURCES} )

target_include_directories(${PROJECT_NAME} PUBLIC
	$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src>
	${PNG_INCLUDE_DIRS} ${FREETYPE_INCLUDE_DIRS} ${FREETYPE_INCLUDE_DIRS}/freetype
	${ALUT_INCLUDE_DIRS} ${X11_INCLUDE_DIRS} ${XRANDR_INCLUDE_DIRS}
	${Vorbis_INCLUDE_DIRS} ${SDL_INCLUDE_DIRS} ${IRRLICHT_INCLUDE_DIR})

target_link_libraries(${PROJECT_NAME}
	${PNG_LIBRARIES} ${Vorbis_LIBRARIES} 
	${FREETYPE_LIBRARIES} ${ALUT_LIBRARIES}
	${X11_LIBRARIES} ${XRANDR_LIBARY} OpenGL
	${SDL_LIBRARIES} ${IRRLICHT_LIBRARY})

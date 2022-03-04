cmake_minimum_required(VERSION 3.0)

project(supertux)
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

file(GLOB SOURCES
	src/*.h
	src/*.cpp
)

option(DEBUG "Build with debug symbols" OFF)
option(NOOPENGL "Disable OpenGL support" OFF)

if (DEBUG)
	set(CMAKE_BUILD_TYPE Debug)
endif()

if (WIN32)
	set(SDL2_LIBRARY ${PROJECT_SOURCE_DIR}/windows/SDL2/lib)
	set(SDL2_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/windows/SDL2/include)
	set(SDL2_IMAGE_LIBRARY ${PROJECT_SOURCE_DIR}/windows/SDL2/lib)
	set(SDL2_IMAGE_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/windows/SDL2/include)
	set(SDL2_MIXER_LIBRARY ${PROJECT_SOURCE_DIR}/windows/SDL2/lib)
	set(SDL2_MIXER_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/windows/SDL2/include)

	file(GLOB ZLIB_SRC
		windows/zlib/*.h
		windows/zlib/*.c
	)
	include_directories(windows/zlib)
	add_library(z STATIC ${ZLIB_SRC})

	link_directories(${SDL2_LIBRARY})
	link_directories(${SDL2_IMAGE_LIBRARY})
	link_directories(${SDL2_MIXER_LIBRARY})
else()
	find_package(ZLIB REQUIRED)
endif()

find_package(SDL2 REQUIRED)
find_package(SDL2_image REQUIRED)
find_package(SDL2_mixer REQUIRED)
include_directories(${SDL2_INCLUDE_DIR})
include_directories(${SDL2_IMAGE_INCLUDE_DIR})
include_directories(${SDL2_MIXER_INCLUDE_DIR})

add_executable(supertux ${SOURCES})

if (WIN32)
	target_link_libraries(supertux opengl32)
	target_compile_definitions(supertux PRIVATE WINDOWS)
else()
	target_link_libraries(supertux GL)
endif()

target_link_libraries(supertux z SDL2_mixer SDL2_image SDL2)
if(NOOPENGL)
	target_compile_definitions(supertux PRIVATE NOOPENGL)
endif()

add_custom_command(TARGET supertux POST_BUILD
                   COMMAND ${CMAKE_COMMAND} -E copy_directory
                       ${PROJECT_SOURCE_DIR}/data $<TARGET_FILE_DIR:supertux>/data)
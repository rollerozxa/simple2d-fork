cmake_minimum_required(VERSION 3.14)
project(simple2d C)
set(CMAKE_C_STANDARD 99)

find_package(SDL2 REQUIRED)

find_package(PkgConfig REQUIRED)
pkg_check_modules(SDL2_image REQUIRED SDL2_image)
pkg_check_modules(SDL2_mixer REQUIRED SDL2_mixer)
pkg_check_modules(SDL2_ttf REQUIRED SDL2_ttf)

set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)

include_directories(
	include/
	${SDL2_INCLUDE_DIRS}
	${SDL2_image_INCLUDE_DIRS}
	${SDL2_mixer_INCLUDE_DIRS}
	${SDL2_ttf_INCLUDE_DIRS})

set(LIBS
	${SDL2_LIBRARIES}
	${SDL2_image_LIBRARIES}
	${SDL2_mixer_LIBRARIES}
	${SDL2_ttf_LIBRARIES}
	${OPENGL_LIBRARIES}
	m)

file(GLOB simple2d_sources
	src/*.c)

set(simple2d_public_headers
	include/simple2d.h)

add_library(${PROJECT_NAME} ${simple2d_sources} ${simple2d_public_headers})

target_include_directories(${PROJECT_NAME} PUBLIC include/)
target_link_libraries(${PROJECT_NAME} ${LIBS})

option(SIMPLE2D_TESTS "Build test programs" FALSE)

if(SIMPLE2D_TESTS)
	add_executable(testcard test/testcard.c)
	add_executable(triangle test/triangle.c)

	target_link_libraries(testcard ${LIBS} simple2d)
	target_link_libraries(triangle ${LIBS} simple2d)
endif()

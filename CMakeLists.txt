# CMakeList.txt for debugging/testing only.
#
# To install from sources, use setup.py to build it
# To build wheels, use build_maylinux_wheels.sh


cmake_minimum_required(VERSION 3.6)

cmake_policy(SET CMP0072 NEW)

project(bimpy)

option (GLFW_BUILD_EXAMPLES OFF)
option (GLFW_BUILD_TESTS OFF)
option (GLFW_INSTALL OFF)
option (USE_MSVC_RUNTIME_LIBRARY_DLL OFF)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared libraries" FORCE)

set(CMAKE_CXX_STANDARD 14)

if(NOT MSVC)
    set (CMAKE_C_FLAGS "-fPIC -g  ${CMAKE_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "-fPIC -g  ${CMAKE_CXX_FLAGS}")
endif()

find_package(OpenGL)

if (OPENGL_FOUND)
	message("OpenGL Correctly Found")
	include_directories(${OPENGL_INCLUDE_DIR})
else (OPENGL_FOUND)
	message("OpenGL environment missing")
endif (OPENGL_FOUND)

find_package (Python COMPONENTS Interpreter Development REQUIRED)

##############################################################
# Generating OpenGL profile
##############################################################
message("Executing gl3w_gen.py")
execute_process(COMMAND python gl3w_gen.py --root ${PROJECT_BINARY_DIR}/gl3w WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/gl3w)


##############################################################
# Externals
##############################################################
add_subdirectory (glfw)
add_subdirectory (gl3w)


#####################################################################
# imgui
#####################################################################
set(IMGUI_DIR imgui)
file(GLOB IMGUI_SOURCES ${IMGUI_DIR}/*.cpp ${IMGUI_DIR}/*.h )
add_library(imgui ${IMGUI_SOURCES})


##############################################################
# Includes
##############################################################
include_directories(glfw/include)
include_directories(pybind11/include)
include_directories(imgui)
include_directories(${Python_INCLUDE_DIRS})

if (OPENGL_FOUND)
	message("OpenGL Correctly Found")
	include_directories(${OPENGL_INCLUDE_DIR})
else (OPENGL_FOUND)
	message("OpenGL environment missing")
endif (OPENGL_FOUND)

##############################################################
# Targets
##############################################################
file(GLOB SOURCES *.cpp *.h )
add_library(bimpy SHARED ${SOURCES})
##############################################################
# Linkage
##############################################################
target_link_libraries(bimpy glfw gl3w imgui ${OPENGL_LIBRARIES} ${Python_LIBRARIES})

SET_TARGET_PROPERTIES(bimpy PROPERTIES PREFIX "_")

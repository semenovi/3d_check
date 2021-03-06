cmake_minimum_required(VERSION 3.8)

set(PROJECT_NAME 3d-check)

project(${PROJECT_NAME})

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

find_package(OpenCL REQUIRED)

FIND_PATH(OPENCL_INCLUDE_DIRS CL/cl.h)
FIND_PATH(_OPENCL_CPP_INCLUDE_DIRS CL/cl.hpp)

FIND_PATH(GLEW_INCLUDE_DIRS GL/glew.h)
FIND_PATH(FREEGLUT_INCLUDE_DIRS GL/glut.h)

FIND_PATH(GLEW_LIB_DIRS glew32.lib)
FIND_PATH(FREEGLUT_LIB_DIRS freeglut.lib)

link_directories(${GLEW_LIB_DIRS} ${FREEGLUT_LIB_DIRS})

FIND_LIBRARY(GLEW_LIBRARIES glew32.lib)
FIND_LIBRARY(FREEGLUT_LIBRARIES freeglut.lib)

# The AMD SDK currently installs both x86 and x86_64 libraries
# This is only a hack to find out architecture
IF( ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "AMD64" )
	SET(OPENCL_LIB_DIR "$ENV{ATISTREAMSDKROOT}/lib/x86_64")
	SET(OPENCL_LIB_DIR "$ENV{ATIINTERNALSTREAMSDKROOT}/lib/x86_64")
ELSE (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "AMD64")
	SET(OPENCL_LIB_DIR "$ENV{ATISTREAMSDKROOT}/lib/x86")
	SET(OPENCL_LIB_DIR "$ENV{ATIINTERNALSTREAMSDKROOT}/lib/x86")
ENDIF( ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "AMD64" )

# find out if the user asked for a 64-bit build, and use the corresponding 
# 64 or 32 bit NVIDIA library paths to the search:
STRING(REGEX MATCH "Win64" ISWIN64 ${CMAKE_GENERATOR})
IF("${ISWIN64}" STREQUAL "Win64") 
	FIND_LIBRARY(OPENCL_LIBRARIES OpenCL.lib $ENV{CUDA_PATH}/lib/x64 ${OPENCL_LIB_DIR} )
ELSE("${ISWIN64}" STREQUAL "Win64") 
	FIND_LIBRARY(OPENCL_LIBRARIES OpenCL.lib $ENV{CUDA_PATH}/lib/Win32 ${OPENCL_LIB_DIR} )
ENDIF("${ISWIN64}" STREQUAL "Win64") 

GET_FILENAME_COMPONENT(_OPENCL_INC_CAND ${OPENCL_LIB_DIR}/../../include ABSOLUTE)

# On Win32 search relative to the library
FIND_PATH(OPENCL_INCLUDE_DIRS CL/cl.h PATHS "${_OPENCL_INC_CAND}" $ENV{CUDA_INC_PATH} $ENV{CUDA_PATH}/include)
FIND_PATH(_OPENCL_CPP_INCLUDE_DIRS CL/cl.hpp PATHS "${_OPENCL_INC_CAND}" $ENV{CUDA_INC_PATH} $ENV{CUDA_PATH}/include)

set(TARGET_SRC
	main.cpp
	kernel.cl
	)

set(TARGET_HEADERS
	OBJ_Loader.h
	)

add_executable(${PROJECT_NAME} ${TARGET_SRC} ${TARGET_HEADERS})
target_link_libraries(${PROJECT_NAME} ${OPENCL_LIBRARIES} ${GLEW_LIBRARIES} ${FREEGLUT_LIBRARIES})

include_directories(${OPENCL_INCLUDE_DIRS} ${GLEW_INCLUDE_DIRS} ${FREEGLUT_INCLUDE_DIRS})

include_directories("${CMAKE_SOURCE_DIR}/khronos")

configure_file(kernel.cl ${CMAKE_CURRENT_BINARY_DIR}/kernel.cl COPYONLY)
configure_file(box_stack.obj ${CMAKE_CURRENT_BINARY_DIR}/box_stack.obj COPYONLY)
configure_file(dll/freeglut.dll ${CMAKE_CURRENT_BINARY_DIR}/freeglut.dll COPYONLY)
configure_file(dll/glew32.dll ${CMAKE_CURRENT_BINARY_DIR}/glew32.dll COPYONLY)

set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
 ################################### The zlib License #####################################
 #
 # Copyright (c) 2020 Halo Development Team
 #
 # This software is provided 'as-is', without any express or implied
 # warranty. In no event will the authors be held liable for any damages
 # arising from the use of this software.
 #
 # Permission is granted to anyone to use this software for any purpose,
 # including commercial applications, and to alter it and redistribute it
 # freely, subject to the following restrictions:
 #
 # 1. The origin of this software must not be misrepresented; you must not
 # claim that you wrote the original software. If you use this software
 # in a product, an acknowledgment in the product documentation would be
 # appreciated but is not required.
 #
 # 2. Altered source versions must be plainly marked as such, and must not be
 # misrepresented as being the original software.
 #
 # 3. This notice may not be removed or altered from any source
 # distribution.
 #
 ##########################################################################################

cmake_minimum_required (VERSION 3.2)
project(Halo)
set (Halo_VERSION_MAJOR 5)
set (Halo_VERSION_MINOR 9)
set (Halo_VERSION_PATCH 0)

set(CMAKE_VERBOSE_MAKEFILE OFF)

find_package(Vulkan REQUIRED)

#https://stackoverflow.com/questions/2152077/is-it-possible-to-get-cmake-to-build-both-a-static-and-shared-version-of-the-sam/29824424#29824424
#TODO: build both static and shared library at the same time
option(Halo_BUILD_STATIC "Build Halo as a static library. Otherwise it is built as a shared library." ON)

# http://www.openguru.com/2009/04/cmake-detecting-platformoperating.html
IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
   message("Building for Mac OS X")
   set(OperatingSystem "OSX")
   set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/mac)
   set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/mac)
   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/mac)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

IF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
   message("Building for Linux")
   set(OperatingSystem "Linux")
   set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/linux)
   set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/linux)
   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/linux)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

IF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
   message("Building for Windows")
   set(OperatingSystem "Windows")
   set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/win)
   set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/win)
   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/win)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")


# Include headers don't need to be added to the compilation source
#  but some IDEs will only show them if they are, so I will GLOB them
file ( GLOB INC common/include/*.h common/include/*.hpp common/src/*.h common/src/*.hpp )

set( CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules" )

set ( SRC	${SRC}
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/VulkanRender.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/VulkanTextureBuilder.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderCullingVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderObject2dVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderObject3dVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderPrimitive2dVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderText2dVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderTransform2dVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderTransform3dVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/render/vulkan/RenderTransformCommonVulkan.cpp
			${CMAKE_CURRENT_SOURCE_DIR}/common/src/platform/OSVulkanManager.cpp
	) 

include_directories( "${CMAKE_CURRENT_SOURCE_DIR}/common/include"
					 "${CMAKE_CURRENT_SOURCE_DIR}/common/src"
					 "${CMAKE_CURRENT_SOURCE_DIR}/common"
					 SDL2_INCLUDE_DIR
					 TINYXML_INCLUDE_DIR
					)

set (SDL_SHARED OFF)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/common/dependencies/SDL-2.0)
set (SDL2_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/common/dependencies/SDL-2.0/include)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/common/dependencies/tinyxml)
set ( TINYXML_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/common/dependencies/tinyxml)

if (Halo_BUILD_STATIC)
	set (Halo_BUILD_TYPE STATIC)
else ()
	set (Halo_BUILD_TYPE SHARED)
endif (Halo_BUILD_STATIC)


add_definitions("-fPIC")

add_library( Halo ${Halo_BUILD_TYPE} ${SRC} ${INC} )

target_link_libraries( Halo TinyXML SDL2 )

#https://foonathan.net/blog/2018/10/17/cmake-warnings.html
target_include_directories(Halo PRIVATE include/
                                SYSTEM INTERFACE include/)




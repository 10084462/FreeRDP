# WinPR: Windows Portable Runtime
# winpr cmake build script
#
# Copyright 2012 Marc-Andre Moreau <marcandre.moreau@gmail.com>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

if (APPLE)
	# flat_namespace should be avoided, but is required for -undefined warning. Since WinPR currently has
	# a lot of undefined symbols in use, use this hack until they're filled out.
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-flat_namespace,-undefined,warning")	
endif()

set(WINPR_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(WINPR_SRCS "")
set(WINPR_LIBS "")
set(WINPR_INCLUDES "")
set(WINPR_DEFINITIONS "")

macro (winpr_module_add)
    file (RELATIVE_PATH _relPath "${WINPR_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}")
    foreach (_src ${ARGN})
        if (_relPath)
            list (APPEND WINPR_SRCS "${_relPath}/${_src}")
        else()
            list (APPEND WINPR_SRCS "${_src}")
        endif()
    endforeach()
    if (_relPath)
        set (WINPR_SRCS ${WINPR_SRCS} PARENT_SCOPE)
    endif()
endmacro()

macro (winpr_include_directory_add)
    file (RELATIVE_PATH _relPath "${WINPR_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}")
    foreach (_inc ${ARGN})
				if (IS_ABSOLUTE ${_inc})
							list (APPEND WINPR_INCLUDES "${_inc}")
				else()
					if (_relPath)
							list (APPEND WINPR_INCLUDES "${_relPath}/${_inc}")
					else()
							list (APPEND WINPR_INCLUDES "${_inc}")
					endif()
				endif()
    endforeach()
    if (_relPath)
        set (WINPR_INCLUDES ${WINPR_INCLUDES} PARENT_SCOPE)
    endif()
endmacro()

macro (winpr_library_add)
    foreach (_lib ${ARGN})
            list (APPEND WINPR_LIBS "${_lib}")
    endforeach()
     set (WINPR_LIBS ${WINPR_LIBS} PARENT_SCOPE)
endmacro()

macro (winpr_definition_add)
    foreach (_define ${ARGN})
            list (APPEND WINPR_DEFINITIONS "${_define}")
    endforeach()
     set (WINPR_DEFINITIONS ${WINPR_DEFINITIONS} PARENT_SCOPE)
endmacro()

# Level "1" API as defined for MinCore.lib
set(WINPR_CORE synch locale library file comm pipe interlocked security 
	environment crypto registry credentials path io memory input 
	heap utils error com timezone sysinfo pool handle thread) 

foreach(DIR ${WINPR_CORE})
	add_subdirectory(${DIR})
endforeach()

set(WINPR_LEVEL2 winsock sspi winhttp asn1 sspicli crt bcrypt rpc credui 
	wtsapi dsparse wnd smartcard nt)

foreach(DIR ${WINPR_LEVEL2})
	add_subdirectory(${DIR})
endforeach()

set(MODULE_NAME winpr)
list(REMOVE_DUPLICATES WINPR_DEFINITIONS)
list(REMOVE_DUPLICATES WINPR_LIBS)
list(REMOVE_DUPLICATES WINPR_INCLUDES)
include_directories(${WINPR_INCLUDES})
add_library(${MODULE_NAME} ${WINPR_SRCS})
set_target_properties(${MODULE_NAME} PROPERTIES LINKER_LANGUAGE C)
if (WITH_LIBRARY_VERSIONING)
 	set_target_properties(${MODULE_NAME} PROPERTIES VERSION ${WINPR_VERSION_FULL} SOVERSION ${WINPR_VERSION})
endif()
set_target_properties(${MODULE_NAME} PROPERTIES PREFIX "lib")
add_definitions(${WINPR_DEFINITIONS})
target_link_libraries(${MODULE_NAME} ${WINPR_LIBS})
install(TARGETS ${MODULE_NAME} DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT libraries EXPORT WinPRTargets)
set_property(TARGET ${MODULE_NAME} PROPERTY FOLDER "WinPR/libwinpr")

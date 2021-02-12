CMAKE_MINIMUM_REQUIRED(VERSION 3.15)

LIST( APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMake )

SET_TARGET_PROPERTIES ( ${PROJECT_EXECUTABLES} PROPERTIES
	LINKER_LANGUAGE CXX
)

#INCLUDE_DIRECTORIES (
#	${CMAKE_CURRENT_BINARY_DIR}/include/rbdl
#	${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/
#)

SET ( URDFREADER_SOURCES
	urdfreader.cc
	)
find_package(urdfdom REQUIRED)
#SET (URDFREADER_DEPENDENCIES
#	rbdl
##	${urdf_LIBRARIES}
#	)

ADD_EXECUTABLE (rbdl_urdfreader_util rbdl_urdfreader_util.cc)

# static library
add_library(rbdl_urdfreader-static STATIC ${URDFREADER_SOURCES} )
set_target_properties(rbdl_urdfreader-static PROPERTIES 
POSITION_INDEPENDENT_CODE ON)

TARGET_LINK_LIBRARIES (rbdl_urdfreader-static
	rbdl-static
	${urdfdom_headers_TARGETS}
	${urdfdom_TARGETS}
	)

TARGET_LINK_LIBRARIES (rbdl_urdfreader_util
	rbdl_urdfreader-static
	rbdl-static
	${urdfdom_headers_TARGETS}
	${urdfdom_TARGETS}
	)

INSTALL (TARGETS rbdl_urdfreader_util rbdl_urdfreader-static EXPORT rbdl
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	ARCHIVE 
		DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# Shared library
add_library (rbdl_urdfreader-shared SHARED ${URDFREADER_SOURCES} )
SET_TARGET_PROPERTIES (rbdl_urdfreader-shared PROPERTIES
	VERSION ${RBDL_VERSION}
	SOVERSION ${RBDL_SO_VERSION}
	POSITION_INDEPENDENT_CODE ON
)

TARGET_LINK_LIBRARIES (rbdl_urdfreader-shared
	rbdl-shared
	${urdfdom_headers_TARGETS}
	${urdfdom_TARGETS}
	)

# If an older version of urdfdom_headers is used such that the urdfdom_headers_TARGETS is not defined
if(NOT DEFINED urdfdom_headers_TARGETS)
	message(WARNING "urdfdom_headers_TARGETS not found, urdfdom_headers version: ${urdfdom_headers_VERSION}. 
	Using fallback to target_include_directories on variable urdfdom_headers_INCLUDE_DIRS which has value of ${urdfdom_headers_INCLUDE_DIRS}")
	target_include_directories(rbdl_urdfreader-static PUBLIC ${urdfdom_headers_INCLUDE_DIRS})
	target_include_directories(rbdl_urdfreader-shared PUBLIC ${urdfdom_headers_INCLUDE_DIRS})
endif(NOT DEFINED urdfdom_headers_TARGETS)
# If an older version of urdfdom is used such that the urdfdom_TARGETS is not defined
if(NOT DEFINED urdfdom_TARGETS)
	message(WARNING "urdfdom_TARGETS not found. 
	Using fallback to target_include directories on variable urdfdom_INCLUDE_DIRS which has value of ${urdfdom_INCLUDE_DIRS}
	Using fallback to target_link_libraries on variable urdfdom_LIBRARIES which has value of ${urdfdom_LIBRARIES}")
	target_include_directories(rbdl_urdfreader-static PUBLIC ${urdfdom_INCLUDE_DIRS})
	target_link_libraries(rbdl_urdfreader-static ${urdfdom_LIBRARIES})
	target_include_directories(rbdl_urdfreader-shared PUBLIC ${urdfdom_INCLUDE_DIRS})
	target_link_libraries(rbdl_urdfreader-shared ${urdfdom_LIBRARIES})
	target_include_directories(rbdl_urdfreader_util PUBLIC ${urdfdom_INCLUDE_DIRS})
	target_link_libraries(rbdl_urdfreader_util ${urdfdom_LIBRARIES})
endif(NOT DEFINED urdfdom_TARGETS)


# get_cmake_property(_variableNames VARIABLES)
# list (SORT _variableNames)
# foreach (_variableName ${_variableNames})
# 	message(WARNING "${_variableName}=${${_variableName}}")
# endforeach()
	


INSTALL (TARGETS rbdl_urdfreader-shared EXPORT rbdl
	LIBRARY 
		DESTINATION ${CMAKE_INSTALL_LIBDIR}
	INCLUDES DESTINATION include
	)



FILE ( GLOB headers
	"${CMAKE_CURRENT_SOURCE_DIR}/*.h"
	)

INSTALL ( FILES ${headers} 
	DESTINATION
	${CMAKE_INSTALL_INCLUDEDIR}/rbdl/addons/urdfreader 
	)

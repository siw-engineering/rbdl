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
find_package(urdfdom 2.3.3 REQUIRED)

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

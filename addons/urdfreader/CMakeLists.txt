CMAKE_MINIMUM_REQUIRED(VERSION 3.11)

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

IF (RBDL_BUILD_STATIC)
	ADD_LIBRARY ( rbdl_urdfreader-static STATIC ${URDFREADER_SOURCES} )

	IF (NOT WIN32)
		SET_TARGET_PROPERTIES ( rbdl_urdfreader-static PROPERTIES PREFIX "lib")
	ENDIF (NOT WIN32)
	SET_TARGET_PROPERTIES ( rbdl_urdfreader-static PROPERTIES OUTPUT_NAME "rbdl_urdfreader")

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

	INSTALL (TARGETS rbdl_urdfreader-static rbdl_urdfreader_util
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  	ARCHIVE DESTINATION ${CMAKE_INSTALL_BINDIR}
	)
ELSE (RBDL_BUILD_STATIC)
	ADD_LIBRARY ( rbdl_urdfreader SHARED ${URDFREADER_SOURCES} )
	SET_TARGET_PROPERTIES ( rbdl_urdfreader PROPERTIES
		VERSION ${RBDL_VERSION}
		SOVERSION ${RBDL_SO_VERSION}
  )

	TARGET_LINK_LIBRARIES (rbdl_urdfreader
		rbdl
		${urdfdom_headers_TARGETS}
		${urdfdom_TARGETS}
		)

	TARGET_LINK_LIBRARIES (rbdl_urdfreader_util
		rbdl_urdfreader
		${urdfdom_headers_TARGETS}
		${urdfdom_TARGETS}
		)

	INSTALL (TARGETS rbdl_urdfreader rbdl_urdfreader_util EXPORT rbdl_urdfreader
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		INCLUDES DESTINATION include
		)
	install(EXPORT rbdl_urdfreader
			FILE rbdl_urdfreaderConfig.cmake
			DESTINATION ${CMAKE_INSTALL_PREFIX})
ENDIF (RBDL_BUILD_STATIC)

FILE ( GLOB headers
	"${CMAKE_CURRENT_SOURCE_DIR}/*.h"
	)

INSTALL ( FILES ${headers} 
	DESTINATION
	${CMAKE_INSTALL_INCLUDEDIR}/rbdl/addons/urdfreader 
	)

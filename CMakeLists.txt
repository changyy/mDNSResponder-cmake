CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
PROJECT (mDNS)

IF(DEFINED CMAKE_TOOLCHAIN_FILE)
	MESSAGE("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	GET_FILENAME_COMPONENT(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	GET_FILENAME_COMPONENT(COMPILER_PATH ${CMAKE_C_COMPILER} PATH)
	#MESSAGE( "CMAKE_C_COMPILER: ${COMPILER_PATH}/${COMPILER_NAME} ")
	STRING(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	MESSAGE( "target_platform: ${target_platform}")
	SET( COMPILER_SYSROOT_LIB_PATH "${COMPILER_PATH}/../sysroot/lib" )
	message( "[INFO] COMPILER_NAME: ${COMPILER_NAME} " )
	message( "[INFO] COMPILER_SYSROOT_LIB_PATH: ${COMPILER_SYSROOT_LIB_PATH} " )
	SET( COMPILER_ADD_LIBS "${COMPILER_SYSROOT_LIB_PATH}/libgcc_s.so.1;${COMPILER_SYSROOT_LIB_PATH}/ld-linux.so.3;${COMPILER_SYSROOT_LIB_PATH}/libc.so.6" )
	message( "[INFO] USE_LIB(replace ' ' to ';'): ${COMPILER_ADD_LIBS} " )
ENDIF(DEFINED CMAKE_TOOLCHAIN_FILE)

# Output
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# MDNS
#SET(MDNS_TARBALL_SOURCE mDNSResponder-320.10.80.tar.gz)
#SET(MDNS_TARBALL_DIR mDNSResponder-320.10.80)

SET(MDNS_TARBALL_SOURCE mDNSResponder-379.37.tar.gz)
SET(MDNS_TARBALL_DIR mDNSResponder-379.37)
SET(MDNS_TARBALL_PATCH mDNSResponder-379.37.patch)

IF(DEFINED CMAKE_TOOLCHAIN_FILE)
	ADD_CUSTOM_COMMAND(
		OUTPUT	BUILD_MDNS
		COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${MDNS_TARBALL_SOURCE} || wget http://www.opensource.apple.com/tarballs/mDNSResponder/${MDNS_TARBALL_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${MDNS_TARBALL_SOURCE}
		COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${MDNS_TARBALL_SOURCE}
		COMMAND cd ${MDNS_TARBALL_DIR} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${MDNS_TARBALL_PATCH}
		#COMMAND cd ${MDNS_TARBALL_DIR}/mDNSPosix && make os=linux HAVE_IPV6=0 LINKOPTS="${COMPILER_ADD_LIBS}" CC=${target_platform}-gcc AR=${target_platform}-ar LD=${target_platform}-ld RANLIB=${target_platform}-ranlib NM=${target_platform}-nm AS=${target_platform}-as OBJDUMP=${target_platform}-objdump OBJCOPY=${target_platform}-objcopy STRIP=${target_platform}-strip STRING=${target_platform}-strings SIZE=${target_platform}-size CPP=${target_platform}-cpp 
		COMMAND cd ${MDNS_TARBALL_DIR}/mDNSPosix && make os=linux HAVE_IPV6=0 LINKOPTS="${COMPILER_ADD_LIBS}" CC=${target_platform}-gcc AR=${target_platform}-ar RANLIB=${target_platform}-ranlib NM=${target_platform}-nm AS=${target_platform}-as OBJDUMP=${target_platform}-objdump OBJCOPY=${target_platform}-objcopy STRIP=${target_platform}-strip STRING=${target_platform}-strings SIZE=${target_platform}-size CPP=${target_platform}-cpp 
		COMMAND mkdir -p ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} && cp ${MDNS_TARBALL_DIR}/mDNSPosix/build/prod/mDNSClientPosix ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/mdns_client && cp ${MDNS_TARBALL_DIR}/mDNSPosix/build/prod/mDNSResponderPosix ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/mdns_responder && cp ${MDNS_TARBALL_DIR}/Clients/build/dns-sd ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/mdns_dns-sd
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT BUILD_MDNS
	)
ELSE(DEFINED CMAKE_TOOLCHAIN_FILE)
	ADD_CUSTOM_COMMAND(
		OUTPUT	BUILD_MDNS
		COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${MDNS_TARBALL_SOURCE} || wget http://www.opensource.apple.com/tarballs/mDNSResponder/${MDNS_TARBALL_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${MDNS_TARBALL_SOURCE}
		COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${MDNS_TARBALL_SOURCE}
		COMMAND cd ${MDNS_TARBALL_DIR}/mDNSPosix && make os=linux
		COMMAND mkdir -p ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} && cp ${MDNS_TARBALL_DIR}/mDNSPosix/build/prod/mDNSClientPosix ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/mdns_client && cp ${MDNS_TARBALL_DIR}/mDNSPosix/build/prod/mDNSResponderPosix ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/mdns_responder && cp ${MDNS_TARBALL_DIR}/Clients/build/dns-sd ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/mdns_dns-sd
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT BUILD_MDNS
	)
ENDIF(DEFINED CMAKE_TOOLCHAIN_FILE)

add_custom_target(build_mdns ALL DEPENDS BUILD_MDNS)

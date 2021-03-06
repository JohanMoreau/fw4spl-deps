CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(cppunitBuilder)

INCLUDE(ExternalProject)

GET_CACHED_URL(http://freefr.dl.sourceforge.net/cppunit/cppunit-1.12.1.tar.gz CACHED_URL)

IF(WIN32)
    SET(CPPUNIT_PATCH_DIR ${CMAKE_CURRENT_SOURCE_DIR}/patch)
    SET(CPPUNIT_CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} 
                           -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
                           -DBUILD_SHARED_LIBS:BOOL=ON
    )
    ExternalProject_Add(
        cppunit
        URL ${CACHED_URL}
        DOWNLOAD_DIR ${ARCHIVE_DIR}
        PATCH_COMMAND ${CMAKE_COMMAND} -E  copy_if_different "${CPPUNIT_PATCH_DIR}/CMakeLists.txt" "<SOURCE_DIR>/CMakeLists.txt"
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
        CMAKE_ARGS ${CPPUNIT_CMAKE_ARGS}
	STEP_TARGETS CopyConfigFileToInstall
    )
ELSE()
    set(FIX_CONFIGURE)
    IF(APPLE)
        # cppunit's configure return an error on osx, but configure worked well
        set(FIX_CONFIGURE || echo fix...)
    ENDIF()

    SET(PATCH_COMMAND patch -t -N -p0 < ${CMAKE_CURRENT_SOURCE_DIR}/patch/demangle.patch && autoconf )
    ExternalProject_Add(
        cppunit
        URL ${CACHED_URL}
        DOWNLOAD_DIR ${ARCHIVE_DIR}
        BUILD_IN_SOURCE 1
        PATCH_COMMAND ${PATCH_COMMAND}
        CONFIGURE_COMMAND <SOURCE_DIR>/configure
                        --enable-shared
                        --disable-static
                        LDFLAGS=-ldl
                        --prefix=${CMAKE_INSTALL_PREFIX}
                        ${FIX_CONFIGURE}
        BUILD_COMMAND ${MAKE} -f <SOURCE_DIR>/Makefile
        INSTALL_COMMAND ${MAKE} -f <SOURCE_DIR>/Makefile install
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
	STEP_TARGETS CopyConfigFileToInstall 
   )
ENDIF()

ExternalProject_Add_Step(cppunit CopyConfigFileToInstall
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/cmake/findBinpkgs/FindCppUnit.cmake ${CMAKE_INSTALL_PREFIX}/FindCppUnit.cmake
    COMMENT "Install configuration file"
)

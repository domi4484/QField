#############################################################
# CMake settings
CMAKE_MINIMUM_REQUIRED(VERSION 3.5)


SET(CMAKE_COLOR_MAKEFILE ON)
SET (APP_NAME "QField" CACHE STRING "Application Name")
SET (APK_VERSION_CODE "1" CACHE STRING "Apk Version Code (Example: 1)")
SET (APP_VERSION "" CACHE STRING "Application Version (Example: 1.0.0)")
SET (APP_VERSION_STR "local - dev" CACHE STRING "Application Version Name (Example: 1.0.0 - Homerun)")
SET (APP_PACKAGE_NAME "qfield" CACHE STRING "Package name suffix. E.g. qfield --> ch.opengis.qfield")

STRING(REGEX REPLACE "v" "" CLEAN_APP_VERSION "${APP_VERSION}")

PROJECT(${APP_NAME}
  LANGUAGES CXX
  VERSION ${CLEAN_APP_VERSION}
)

SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

INCLUDE(GetGitRevisionDescription)
GET_GIT_HEAD_REVISION(GIT_REFSPEC GIT_REV)

ADD_DEFINITIONS(-DAPP_NAME="${APP_NAME}")
ADD_DEFINITIONS(-DAPP_VERSION="${APP_VERSION}")
ADD_DEFINITIONS(-DAPP_VERSION_STR="${APP_VERSION_STR}")
ADD_DEFINITIONS(-DGIT_REV="${GIT_REV}")

CMAKE_POLICY(SET CMP0071 NEW)

SET (DEFAULT_BIN_SUBDIR     bin)
SET (DEFAULT_CGIBIN_SUBDIR  bin)
SET (DEFAULT_LIB_SUBDIR     lib${LIB_SUFFIX})
SET (DEFAULT_INCLUDE_SUBDIR include/qfield)

SET (QFIELD_BIN_SUBDIR     ${DEFAULT_BIN_SUBDIR}     CACHE STRING "Subdirectory where executables will be installed")
SET (QFIELD_LIB_SUBDIR     ${DEFAULT_LIB_SUBDIR}     CACHE STRING "Subdirectory where libraries will be installed")
SET (QFIELD_INCLUDE_SUBDIR ${DEFAULT_INCLUDE_SUBDIR} CACHE STRING "Subdirectory where header files will be installed")

MARK_AS_ADVANCED (QFIELD_INCLUDE_SUBDIR QFIELD_BIN_SUBDIR QFIELD_LIB_SUBDIR)
SET (QFIELD_BIN_DIR ${QFIELD_BIN_SUBDIR})
SET (QFIELD_LIB_DIR ${QFIELD_LIB_SUBDIR})
SET (QFIELD_INCLUDE_DIR ${QFIELD_INCLUDE_SUBDIR})
SET (QT_ANDROID_APPLICATION_BINARY "qfield")

# set the default locations where the targets (executables, libraries) will land when compiled
# this is to allow running QField directly from the build directory.
IF (ANDROID)
  SET (QFIELD_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/android-build)
ELSE ()
  SET (QFIELD_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/output)
ENDIF ()

SET (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${QFIELD_OUTPUT_DIRECTORY}/${QFIELD_BIN_SUBDIR})
SET (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${QFIELD_OUTPUT_DIRECTORY}/${QFIELD_LIB_SUBDIR})

install(DIRECTORY ${CMAKE_SOURCE_DIR}/resources/ DESTINATION ${CMAKE_INSTALL_PREFIX}/share/qfield)

IF (ANDROID)
  ADD_DEFINITIONS(-DQGIS_INSTALL_DIR="") # TODO: do we need osgeo4a/[lib]/files here? see qgis.pri
  SET (ANDROID_TARGET_PLATFORM CACHE INT "The target android platform sdk version")
  SET (ANDROID_PACKAGE_SOURCE_DIR ${CMAKE_SOURCE_DIR}/android)
  SET (OSGEO4A_STAGE_DIR NONE CACHE DIRECTORY "OSGEO4A stage path (base path without architecture)")
  SET (OSGEO4A_LIB_DIR ${OSGEO4A_STAGE_DIR}/${ANDROID_ABI}/lib)
  FILE(GLOB ANDROID_EXTRA_LIBS
    ${OSGEO4A_LIB_DIR}/*provider*.so
  )
  LIST(APPEND ANDROID_EXTRA_LIBS
    ${OSGEO4A_LIB_DIR}/libssl_1_1.so
    ${OSGEO4A_LIB_DIR}/libcrypto_1_1.so
    ${OSGEO4A_LIB_DIR}/libexpat.so
    ${OSGEO4A_LIB_DIR}/libgeos.so
    ${OSGEO4A_LIB_DIR}/libgeos_c.so
    ${OSGEO4A_LIB_DIR}/libgslcblas.so
    ${OSGEO4A_LIB_DIR}/libsqlite3.so
    ${OSGEO4A_LIB_DIR}/libcharset.so
    ${OSGEO4A_LIB_DIR}/libexiv2.so
    ${OSGEO4A_LIB_DIR}/libiconv.so
    ${OSGEO4A_LIB_DIR}/libopenjp2.so
    ${OSGEO4A_LIB_DIR}/libfreexl.so
    ${OSGEO4A_LIB_DIR}/libpng16.so
    ${OSGEO4A_LIB_DIR}/libgdal.so
    ${OSGEO4A_LIB_DIR}/libproj.so
    ${OSGEO4A_LIB_DIR}/libspatialindex.so
    ${OSGEO4A_LIB_DIR}/libpq.so
    ${OSGEO4A_LIB_DIR}/libspatialite.so
    ${OSGEO4A_LIB_DIR}/libqca-qt5_${ANDROID_ABI}.so
    ${OSGEO4A_LIB_DIR}/libqgis_core_${ANDROID_ABI}.so
    ${OSGEO4A_LIB_DIR}/libqgis_analysis_${ANDROID_ABI}.so
    ${OSGEO4A_LIB_DIR}/libqgis_native_${ANDROID_ABI}.so
    ${OSGEO4A_LIB_DIR}/libqt5keychain_${ANDROID_ABI}.so
    ${OSGEO4A_LIB_DIR}/libzip.so
    ${OSGEO4A_LIB_DIR}/libzstd.so
    ${OSGEO4A_LIB_DIR}/libpoppler_${ANDROID_ABI}.so
    ${OSGEO4A_LIB_DIR}/libfreetype.so
  )
  FIND_PACKAGE(Qt5 COMPONENTS AndroidExtras REQUIRED)

  CONFIGURE_FILE(cmake/generated.xml.in ${CMAKE_SOURCE_DIR}/android/res/values/generated.xml @ONLY)
  CONFIGURE_FILE(cmake/AndroidManifest.xml.in ${CMAKE_SOURCE_DIR}/android/AndroidManifest.xml @ONLY)

  INCLUDE(CreateZip)

  set(SHARE_PATH "${CMAKE_CURRENT_BINARY_DIR}/android-build/assets/share")
  FILE(COPY "${OSGEO4A_STAGE_DIR}/${ANDROID_ABI}/share/proj/" DESTINATION "${SHARE_PATH}/proj/")

  FILE(COPY "${OSGEO4A_STAGE_DIR}/${ANDROID_ABI}/files/share/svg/" DESTINATION "${SHARE_PATH}/qgis/svg/")
  FILE(COPY "${OSGEO4A_STAGE_DIR}/${ANDROID_ABI}/files/share/resources/" DESTINATION "${SHARE_PATH}/qgis/resources/")
  # Remove world map to keep apk size a bit smaller
  FILE(REMOVE "${SHARE_PATH}/qgis/resources/data/world_map.gpkg")
  FILE(COPY resources/ DESTINATION "${SHARE_PATH}/qfield/")
ENDIF()

SET(QT_MIN_VERSION 5.14.0)
FIND_PACKAGE(Qt5 COMPONENTS Test Concurrent Core Qml Gui Xml Positioning Widgets Network Quick Svg OpenGL Sql Sensors WebView Bluetooth REQUIRED)
# PrintSupport isn't required, because it doesn't exist for ios
# qgis will deal with it an define a public 'QT_NO_PRINTER'
FIND_PACKAGE(Qt5 COMPONENTS PrintSupport QUIET)

FIND_PACKAGE(QGIS REQUIRED)
IF(NOT APPLE)
  FIND_PACKAGE(Proj)
  FIND_PACKAGE(GDAL)
ENDIF()

IF (${CMAKE_SYSTEM_NAME} STREQUAL "iOS")
  ADD_DEFINITIONS(-DQT_NO_PRINTER)

  set(SHARE_PATH "${CMAKE_CURRENT_BINARY_DIR}/share/")

  # keep trailing slash for QField to rename the directory (instead of copying in subdir)
  FILE(COPY ${CMAKE_CURRENT_SOURCE_DIR}/resources/ DESTINATION ${SHARE_PATH}/qfield)
  FILE(COPY ${QGIS_APPLE_RESOURCES_DIR}/resources DESTINATION ${SHARE_PATH}/qgis)
  FILE(COPY ${QGIS_APPLE_RESOURCES_DIR}/svg DESTINATION ${SHARE_PATH}/qgis)
  FILE(COPY ${QFIELD_PROJ_DIR} DESTINATION ${SHARE_PATH})
  # Remove world map to keep apk size a bit smaller
  FILE(REMOVE "${SHARE_PATH}/qgis/resources/data/world_map.gpkg")

  set(QFIELD_RESOURCE_FILES ${SHARE_PATH})
ENDIF()

set (ENABLE_TESTS TRUE CACHE BOOL "Build unit tests?")

ADD_SUBDIRECTORY(3rdparty/tessellate)
ADD_SUBDIRECTORY(src/core)
ADD_SUBDIRECTORY(src/app)
if (ENABLE_TESTS)
  ENABLE_TESTING()
  ADD_SUBDIRECTORY(test)
endif()
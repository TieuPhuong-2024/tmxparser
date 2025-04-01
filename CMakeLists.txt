cmake_minimum_required(VERSION 3.2)

project(tmxparser)

include(GNUInstallDirs)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake ${CMAKE_MODULE_PATH})

set(TMXPARSER_VERSION_MAJOR 2)
set(TMXPARSER_VERSION_MINOR 1)
set(TMXPARSER_VERSION_PATCH 0)

set(TMXPARSER_VERSION "${TMXPARSER_VERSION_MAJOR}.${TMXPARSER_VERSION_MINOR}.${TMXPARSER_VERSION_PATCH}")

message("Configuring tmxparser version ${VERSION}")

configure_file(
  "include/Tmx.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/include/Tmx.h")

option(USE_MINIZ "Use miniz.c instead of zlib (default: OFF)" OFF)
option(BUILD_SHARED_LIBS "Build shared libs. (default: ON)" ON)
option(BUILD_STATIC_LIBS "Build static libs. (default: ON)" ON)
option(BUILD_TINYXML2 "Build tinyxml2 as external project (default: OFF)" OFF)
option(BUILD_TESTS "Build tests. (default: OFF)" OFF)
option(BUILD_DOCS "Build documentation. (default: OFF)" OFF)

#Dependencies Settings
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/deps.cmake)

find_package(tinyxml2 REQUIRED)

if(NOT USE_MINIZ)
  find_package(ZLIB) #<-- build it as external project?
endif()

if(BUILD_SHARED_LIBS)
  add_library(tmxparser SHARED "")
  set_target_properties(tmxparser PROPERTIES
    SOVERSION ${TMXPARSER_VERSION_MAJOR}
    VERSION ${TMXPARSER_VERSION})
endif()

if(BUILD_STATIC_LIBS)
  add_library(tmxparser_static STATIC "")
endif()

set(SRC_FILES
  ${CMAKE_CURRENT_BINARY_DIR}/include/Tmx.h
  src/TmxColor.cpp include/TmxColor.h
  src/TmxEllipse.cpp include/TmxEllipse.h
  src/TmxGroupLayer.cpp include/TmxGroupLayer.h
  src/TmxImage.cpp include/TmxImage.h
  src/TmxImageLayer.cpp include/TmxImageLayer.h
  src/TmxLayer.cpp include/TmxLayer.h
  src/TmxMap.cpp include/TmxMap.h
  src/TmxObject.cpp include/TmxObject.h
  src/TmxObjectGroup.cpp include/TmxObjectGroup.h
  src/TmxPolygon.cpp include/TmxPolygon.h
  src/TmxPolyline.cpp include/TmxPolyline.h
  src/TmxProperty.cpp include/TmxProperty.h
  src/TmxPropertySet.cpp include/TmxPropertySet.h
  src/TmxTerrain.cpp include/TmxTerrain.h
  src/TmxTerrainArray.cpp include/TmxTerrainArray.h
  src/TmxText.cpp include/TmxText.h
  src/TmxTile.cpp include/TmxTile.h
  src/TmxTileset.cpp include/TmxTileset.h
  src/TmxTileLayer.cpp include/TmxTileLayer.h
  src/TmxTileOffset.cpp include/TmxTileOffset.h
  src/TmxUtil.cpp include/TmxUtil.h
  src/base64/base64.cpp include/base64/base64.h
)

if(BUILD_SHARED_LIBS)
  target_sources(tmxparser PRIVATE ${SRC_FILES})
endif()

if(BUILD_STATIC_LIBS)
  target_sources(tmxparser_static PRIVATE ${SRC_FILES})
endif()

if(USE_MINIZ)
  target_sources(tmxparser
    PRIVATE src/miniz.c)
  target_compile_definitions(tmxparser
    PRIVATE -DUSE_MINIZ)
endif()

foreach(TARGET_NAME tmxparser tmxparser_static)
  if(TARGET ${TARGET_NAME})
    target_include_directories(${TARGET_NAME} PUBLIC
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
      $<INSTALL_INTERFACE:include>
      PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>)
  endif()
endforeach()

if(MSVC)
  target_compile_options(tmxparser
    PRIVATE /W3)
else()
  foreach(TARGET_NAME tmxparser tmxparser_static)
    if(TARGET ${TARGET_NAME})
      target_compile_options(${TARGET_NAME}
        PRIVATE -pedantic
        PRIVATE -Wall
        PRIVATE -Werror=strict-prototypes
        PRIVATE -Werror=old-style-definition
        PRIVATE -Werror=missing-prototypes)
    endif()
  endforeach()
  if(NOT USE_MINIZ)
    target_compile_options(tmxparser PRIVATE -Werror)
  endif()
endif()

target_compile_features(tmxparser
  PRIVATE cxx_std_11)

foreach(TARGET_NAME tmxparser tmxparser_static)
  if(TARGET ${TARGET_NAME})
    target_link_libraries(${TARGET_NAME} PRIVATE tinyxml2::tinyxml2)
    if(NOT USE_MINIZ)
      target_link_libraries(${TARGET_NAME} PRIVATE ZLIB::ZLIB)
    endif()
  endif()
endforeach()

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN Tmx.h)

install(DIRECTORY include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

foreach(TARGET_NAME tmxparser tmxparser_static)
  if(TARGET ${TARGET_NAME})
    install(TARGETS ${TARGET_NAME} EXPORT tmxparserTargets
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
      INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
  endif()
endforeach()

install(EXPORT tmxparserTargets
  FILE tmxparserTargets.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tmxparser)

# pkgconfig is okay but export a CMake config file, too
configure_file(src/tmxparser.pc.in tmxparser.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/tmxparser.pc
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

include(CMakePackageConfigHelpers)

configure_package_config_file(cmake/tmxparserConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/tmxparserConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tmxparser
  NO_CHECK_REQUIRED_COMPONENTS_MACRO)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/tmxparserConfigVersion.cmake
  VERSION ${TMXPARSER_VERSION}
  COMPATIBILITY SameMajorVersion)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/tmxparserConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/tmxparserConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tmxparser)

if(BUILD_TESTS)
  add_executable(run_tests test/test.cpp)
  target_compile_options(run_tests
    PRIVATE -pedantic
    PRIVATE -Werror
    PRIVATE -Wall
    PRIVATE -g)
  target_compile_features(run_tests
    PRIVATE cxx_std_11)
  target_link_libraries(run_tests tmxparser tinyxml2)
  file(COPY test/example
    DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
endif()

if(BUILD_DOCS)
  find_package(Doxygen)
  if(DOXYGEN_FOUND)
    configure_file(${PROJECT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
    add_custom_target(doc ALL ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile COMMENT "Generating API documentation with Doxygen" VERBATIM)
  else()
    message(WARNING "Doxygen not found! No documentation was build.")
  endif()
endif()

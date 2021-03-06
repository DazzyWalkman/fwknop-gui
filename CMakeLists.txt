include(FeatureSummary)
cmake_minimum_required(VERSION 2.8)
project(fwknop-gui CXX C)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")
if(NOT(DEFINED IS_WIN32 OR DEFINED IS_LINUX OR DEFINED IS_APPLE))
  if(WIN32)
    set(IS_WIN32 TRUE)
  elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(IS_APPLE TRUE)
    set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR})
  else()
    set(IS_LINUX TRUE)
  endif()
endif()

if (IS_WIN32)
  set( wxWidgets_USE_STATIC ON)
  set(CODE_FILES win32/minimal.rc)
endif()

find_package(wxWidgets 3.0.0 COMPONENTS xml html adv qa richtext net core base REQUIRED)

include(${wxWidgets_USE_FILE})
include_directories(${CMAKE_SOURCE_DIR})

find_package(LibFko REQUIRED)
find_package(CURL REQUIRED)
find_package(LibQrencode REQUIRED)
find_package(Gpgme REQUIRED)

set(CODE_FILES
  ${CODE_FILES}
  configs.cpp
  fwknop_guiApp.cpp
  fwknop_guiMain.cpp
  wizard.cpp
  general_config.cpp
  rc_import.cpp
  rc_export.cpp
  qr_export.cpp
  timer.cpp
  gpgme_wrapper.cpp
  )

if(IS_APPLE)

  set_source_files_properties(MacOS/fwknop-gui.icns PROPERTIES
       MACOSX_PACKAGE_LOCATION "Resources")
  set(CODE_FILES
  ${CODE_FILES}
  MacOS/fwknop-gui.icns
  )
endif()
add_executable(fwknop-gui WIN32 MACOSX_BUNDLE
  ${CODE_FILES}
  )
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

SET_TARGET_PROPERTIES(fwknop-gui PROPERTIES POSITION_INDEPENDENT_CODE ON)

target_link_libraries(fwknop-gui ${wxWidgets_LIBRARIES} ${LIBFKO_LIBRARIES} ${CURL_LIBRARIES} ${LIBQRENCODE_LIBRARIES} ${GPGME_VANILLA_LIBRARIES})
if(IS_LINUX)
find_program(A2X_EXECUTABLE NAMES a2x a2x.py)
set(A2X_OPTS
  -D ${CMAKE_BINARY_DIR}
  -d manpage
  -f manpage
  )

set(MAN_NAMES fwknop-gui.8)
set(MAN_FILES)
foreach(m IN LISTS MAN_NAMES)
  set(mf ${CMAKE_BINARY_DIR}/${m})
  set(ms ${CMAKE_SOURCE_DIR}/${m}.asciidoc)
  add_custom_command(OUTPUT ${mf}
    COMMAND ${A2X_EXECUTABLE} ${A2X_OPTS} ${ms}
    DEPENDS ${ms}
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Building manpage ${mf}"
    VERBATIM)
  list(APPEND MAN_FILES ${mf})
endforeach()

add_custom_target(man ALL DEPENDS ${MAN_FILES})

  install(FILES help.html DESTINATION share/fwknop-gui)
  install(TARGETS fwknop-gui DESTINATION bin)
  install(PROGRAMS fwknop-gui.desktop DESTINATION share/applications)
  install(FILES fwknop-gui.png DESTINATION share/pixmaps)
  INSTALL(FILES fwknop-gui.8 DESTINATION share/man/man8)
endif()
if(IS_WIN32)
  include_directories( ${LIBQRENCODE_INCLUDE_DIR} )
  target_link_libraries(fwknop-gui wsock32 ws2_32 ${wxWidgets_LIBRARIES} ${LIBFKO_LIBRARIES} ${CURL_LIBRARIES} ${GPGME_VANILLA_LIBRARIES})
  set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++ -static")
  include_directories(${LIBQRENCODE_INCLUDE_DIR})
endif()
if(IS_APPLE)
    set(MACOSX_BUNDLE_BUNDLE_NAME fwknop-gui)
    set(MACOSX_BUNDLE_ICON_FILE fwknop-gui.icns)
    set_property(TARGET fwknop-gui PROPERTY MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/MacOS/MacOSXBundleInfo.plist.in")
    INSTALL(CODE "
    set(BU_CHMOD_BUNDLE_ITEMS ON)
    include(BundleUtilities)
    file(MAKE_DIRECTORY ${CMAKE_INSTALL_PREFIX}/fwknop-gui.app/Contents/SharedSupport)
    file(INSTALL ${CMAKE_SOURCE_DIR}/help.html DESTINATION ${CMAKE_INSTALL_PREFIX}/fwknop-gui.app/Contents/SharedSupport/)
    fixup_bundle(${CMAKE_INSTALL_PREFIX}/fwknop-gui.app \"\" \"\")
")
endif()

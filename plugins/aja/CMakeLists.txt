project(aja)

option(ENABLE_AJA "Build OBS with aja support" ON)
if(NOT ENABLE_AJA)
  message(STATUS "OBS:  DISABLED   aja")
  return()
endif()

if(NOT OS_MACOS AND NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
  message(
    STATUS "OBS:             aja support not enabled (32-bit not supported)")
  set(ENABLE_AJA
      OFF
      CACHE BOOL "Build OBS with aja support" FORCE)
  return()
endif()

find_package(LibAJANTV2 REQUIRED)

set(aja_SOURCES
	main.cpp
	aja-card-manager.cpp
	aja-common.cpp
	aja-output.cpp
	aja-presets.cpp
	aja-props.cpp
	aja-routing.cpp
	aja-source.cpp
	aja-vpid-data.cpp
	aja-widget-io.cpp)

set(aja_HEADERS
	aja-card-manager.hpp
	aja-common.hpp
	aja-enums.hpp
	aja-ui-props.hpp
	aja-output.hpp
	aja-presets.hpp
	aja-props.hpp
	aja-routing.hpp
	aja-source.hpp
	aja-vpid-data.hpp
	aja-widget-io.hpp)

target_link_libraries(aja PRIVATE OBS::libobs AJA::LibAJANTV2)

if(TARGET AJA::LibAJANTV2_Debug)
  target_link_libraries(aja PRIVATE debug AJA::LibAJANTV2_Debug)
endif()

if(OS_MACOS)
  find_library(IOKIT IOKit)
  find_library(COREFOUNDATION CoreFoundation)
  find_library(APPKIT AppKit)

  target_link_libraries(aja PRIVATE ${IOKIT} ${COREFOUNDATION} ${APPKIT})
elseif(OS_WINDOWS)
  set(MODULE_DESCRIPTION "OBS AJA Windows module")
  configure_file(${CMAKE_SOURCE_DIR}/cmake/bundle/windows/obs-module.rc.in
                 win-aja.rc)

  target_sources(aja PRIVATE win-aja.rc)

  target_link_libraries(aja PRIVATE ws2_32.lib setupapi.lib Winmm.lib
                                    netapi32.lib Shlwapi.lib)
endif()

set_target_properties(aja PROPERTIES FOLDER "plugins/aja" PREFIX "")

setup_plugin_target(aja)

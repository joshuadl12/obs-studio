project(aja-output-ui)

if(NOT ENABLE_AJA)
  return()
endif()

find_package(LibAJANTV2 REQUIRED)

add_library(aja-output-ui MODULE)
add_library(OBS::aja-output-ui ALIAS aja-output-ui)

find_qt(
  VERSION ${QT_VERSION}
  COMPONENTS Widgets
  COMPONENTS_LINUX X11Extras)

if(OS_POSIX AND NOT OS_MACOS)
  target_link_libraries(aja-output-ui PRIVATE Qt::X11Extras)
endif()

set(aja-output-ui_HEADERS
	${aja-output-ui_HEADERS}
	../../qt-wrappers.hpp
	../../properties-view.hpp
	../../properties-view.moc.hpp
	../../vertical-scroll-area.hpp
	../../double-slider.hpp
	../../slider-ignorewheel.hpp
	../../combobox-ignorewheel.hpp
	../../spinbox-ignorewheel.hpp
	AJAOutputUI.h
	aja-ui-main.h
	../../../plugins/aja/aja-card-manager.hpp
	../../../plugins/aja/aja-common.hpp
	../../../plugins/aja/aja-enums.hpp
	../../../plugins/aja/aja-presets.hpp
	../../../plugins/aja/aja-props.hpp
	../../../plugins/aja/aja-routing.hpp
	../../../plugins/aja/aja-ui-props.hpp
	../../../plugins/aja/aja-vpid-data.hpp
	../../../plugins/aja/aja-widget-io.hpp
	)

set(aja-output-ui_SOURCES
	${aja-output-ui_SOURCES}
	../../qt-wrappers.cpp
	../../properties-view.cpp
	../../vertical-scroll-area.cpp
	../../double-slider.cpp
	../../slider-ignorewheel.cpp
	../../combobox-ignorewheel.cpp
	../../spinbox-ignorewheel.cpp
	AJAOutputUI.cpp
	aja-ui-main.cpp
	../../../plugins/aja/aja-card-manager.cpp
	../../../plugins/aja/aja-common.cpp
	../../../plugins/aja/aja-presets.cpp
	../../../plugins/aja/aja-props.cpp
	../../../plugins/aja/aja-routing.cpp
	../../../plugins/aja/aja-vpid-data.cpp
	../../../plugins/aja/aja-widget-io.cpp
	)

set(aja-output-ui_UI
	${aja-output-ui_UI}
	forms/output.ui)

if(WIN32)
	set(MODULE_DESCRIPTION "OBS AJA Output UI")
	configure_file(${CMAKE_SOURCE_DIR}/cmake/winrc/obs-module.rc.in aja-output-ui.rc)
	list(APPEND aja-output-ui_SOURCES
		aja-output-ui.rc)
endif()

if(APPLE)
	set(aja-output-ui_PLATFORM_LIBS
		${COCOA})
endif()

if(OS_MACOS)
  find_library(IOKIT_FRAMEWORK Iokit)
  find_library(COREFOUNDATION_LIBRARY CoreFoundation)
  find_library(APPKIT_FRAMEWORK AppKit)

  target_link_libraries(aja-output-ui PRIVATE ${IOKIT} ${COREFOUNDATION}
                                              ${APPKIT})
elseif(OS_WINDOWS)
  set(MODULE_DESCRIPTION "OBS AJA Output UI")
  configure_file(${CMAKE_SOURCE_DIR}/cmake/bundle/windows/obs-module.rc.in
                 aja-output-ui.rc)
  target_sources(aja-output-ui PRIVATE aja-output-ui.rc)

  target_link_libraries(aja-output-ui PRIVATE ws2_32.lib setupapi.lib Winmm.lib
                                              netapi32.lib Shlwapi.lib)
else()
  find_package(X11 REQUIRED)
  target_link_libraries(aja-output-ui PRIVATE X11::X11
                                              Qt${QT_VERSION}::GuiPrivate)
endif()

set_target_properties(aja-output-ui PROPERTIES FOLDER "frontend" PREFIX "")

setup_plugin_target(aja-output-ui)

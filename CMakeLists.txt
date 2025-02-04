cmake_minimum_required(VERSION 3.16...3.21)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/Modules")
include(VersionConfig)

# Prohibit in-source builds
if("${CMAKE_BINARY_DIR}" STREQUAL "${CMAKE_SOURCE_DIR}")
  message(
    FATAL_ERROR
      "OBS: You cannot build in a source directory (or any directory with "
      "CMakeLists.txt file). Please make a build subdirectory. Feel free to "
      "remove CMakeCache.txt and CMakeFiles.")
endif()

project(obs-studio VERSION ${OBS_VERSION_CANONICAL})
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Use target folders for MSVC/Xcode/etc.
include(DeprecationHelpers)
include(ObsHelpers)
# Set default compiler flags
include(CompilerConfig)
include(GNUInstallDirs)
include(CheckCCompilerFlag)
include(CheckCXXCompilerFlag)

set(OBS_RELEASE_CANDIDATE_MAJOR 0)
set(OBS_RELEASE_CANDIDATE_MINOR 0)
set(OBS_RELEASE_CANDIDATE_PATCH 0)
set(OBS_RELEASE_CANDIDATE 0)
set(OBS_BETA_MAJOR 0)
set(OBS_BETA_MINOR 0)
set(OBS_BETA_PATCH 0)
set(OBS_BETA 0)

# Must be a string in the format of "x.x.x-rcx" or "x.x.x-betax"
if(DEFINED RELEASE_CANDIDATE)
	set(OBS_VERSION "${RELEASE_CANDIDATE}")
	string(REPLACE "-rc" "." RC_SPLIT ${RELEASE_CANDIDATE})
	string(REPLACE "." ";" RC_SPLIT ${RC_SPLIT})
	message(WARNING "******************************************************************************\nRelease candidate detected, OBS_VERSION is now: ${OBS_VERSION}\n******************************************************************************")
	list(GET RC_SPLIT 0 OBS_RELEASE_CANDIDATE_MAJOR)
	list(GET RC_SPLIT 1 OBS_RELEASE_CANDIDATE_MINOR)
	list(GET RC_SPLIT 2 OBS_RELEASE_CANDIDATE_PATCH)
	list(GET RC_SPLIT 3 OBS_RELEASE_CANDIDATE)
elseif(DEFINED BETA)
	set(OBS_VERSION "${BETA}")
	string(REPLACE "-beta" "." BETA_SPLIT ${BETA})
	string(REPLACE "." ";" BETA_SPLIT ${BETA_SPLIT})
	message(WARNING "******************************************************************************\nBeta detected, OBS_VERSION is now: ${OBS_VERSION}\n******************************************************************************")
	list(GET BETA_SPLIT 0 OBS_BETA_MAJOR)
	list(GET BETA_SPLIT 1 OBS_BETA_MINOR)
	list(GET BETA_SPLIT 2 OBS_BETA_PATCH)
	list(GET BETA_SPLIT 3 OBS_BETA)
endif()

# Binary Versioning for Windows
if(WIN32)
	string(REPLACE "-" ";" UI_VERSION_SPLIT ${OBS_VERSION})
	list(GET UI_VERSION_SPLIT 0 UI_VERSION)
	string(REPLACE "." ";" UI_VERSION_SEMANTIC ${UI_VERSION})
	list(GET UI_VERSION_SEMANTIC 0 UI_VERSION_MAJOR)
	list(GET UI_VERSION_SEMANTIC 1 UI_VERSION_MINOR)
	list(GET UI_VERSION_SEMANTIC 2 UI_VERSION_PATCH)
	set(OBS_COMPANY_NAME "OBS")
	set(OBS_PRODUCT_NAME "OBS Studio")
	set(OBS_COMMENTS "Free and open source software for video recording and live streaming")
	set(OBS_LEGAL_COPYRIGHT "(C) Hugh Bailey")
	set(MODULE_DESCRIPTION "OBS Studio")
	configure_file(UI/obs.rc.in ${PROJECT_BINARY_DIR}/obs.rc)
endif()

if(MSVC AND NOT EXISTS "${CMAKE_BINARY_DIR}/ALL_BUILD.vcxproj.user")
	file(GENERATE
		OUTPUT "${CMAKE_BINARY_DIR}/ALL_BUILD.vcxproj.user"
		INPUT "${CMAKE_SOURCE_DIR}/cmake/ALL_BUILD.vcxproj.user.in")
endif()

# Allow selection of common build types via UI
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE
      "RelWithDebInfo"
      CACHE STRING
            "OBS build type [Release, RelWithDebInfo, Debug, MinSizeRel]" FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Release RelWithDebInfo
                                               Debug MinSizeRel)
endif()

# Global project options
option(BUILD_FOR_DISTRIBUTION "Build for distribution (enables optimisations)"
       OFF)
option(ENABLE_UI "Enable building with UI (requires Qt)" ON)
option(ENABLE_SCRIPTING "Enable scripting support" ON)
option(USE_LIBCXX "Use libc++ instead of libstdc++" ${APPLE})
option(
  BUILD_TESTS
  "Build test directory (includes test sources and possibly a platform test executable)"
  OFF)

if(OS_WINDOWS)
  option(
    INSTALLER_RUN
    "Build a multiarch installer (needs to run independently after both archs have compiled) (Windows)"
    OFF)

elseif(OS_MACOS)
  option(ENABLE_SPARKLE_UPDATER "Enable Sparkle framework for updates (macOS)"
         OFF)

elseif(OS_POSIX)
  option(LINUX_PORTABLE "Build portable version (Linux)" OFF)
  option(USE_XDG "Utilize XDG Base Directory Specification (Linux)" ON)
  if(OS_LINUX)
    option(ENABLE_WAYLAND "Enable building with support for Wayland (Linux)" ON)
    option(BUILD_FOR_PPA "Build for PPA distribution" OFF)
  endif()
endif()

setup_obs_project()
mark_as_advanced(BUILD_TESTS USE_LIBCXX)

if(INSTALLER_RUN)
  generate_multiarch_installer()
  return()
endif()

# OBS sources and plugins
add_subdirectory(deps)
add_subdirectory(libobs-opengl)
if(OS_WINDOWS)
  add_subdirectory(libobs-d3d11)
  add_subdirectory(libobs-winrt)
endif()
add_subdirectory(libobs)
add_subdirectory(plugins)

# OBS main app
add_subdirectory(UI)

# Tests
if(ENABLE_UNIT_TESTS)
  enable_testing()
endif()

if(BUILD_TESTS OR ENABLE_UNIT_TESTS)
  add_subdirectory(test)
endif()

cmake_minimum_required(VERSION 3.22...3.25)

option(ENABLE_AJA "Build OBS with aja support" ON)

if(NOT ENABLE_AJA)
  set_property(GLOBAL APPEND PROPERTY OBS_MODULES_DISABLED aja)
  return()
endif()

find_package(LibAJANTV2 REQUIRED)

add_library(aja MODULE)
add_library(OBS::aja ALIAS aja)

add_library(aja-support OBJECT)
add_library(OBS::aja-support ALIAS aja-support)

target_sources(
  aja-support
  PRIVATE
    aja-card-manager.cpp
    aja-common.cpp
    aja-presets.cpp
    aja-props.cpp
    aja-routing.cpp
    aja-vpid-data.cpp
    aja-widget-io.cpp
  PUBLIC
    aja-card-manager.hpp
    aja-common.hpp
    aja-enums.hpp
    aja-presets.hpp
    aja-props.hpp
    aja-routing.hpp
    aja-ui-props.hpp
    aja-vpid-data.hpp
    aja-widget-io.hpp
)

target_link_libraries(aja-support PUBLIC OBS::libobs AJA::LibAJANTV2)

set_target_properties(aja-support PROPERTIES FOLDER plugins/aja POSITION_INDEPENDENT_CODE TRUE)

target_sources(
  aja
  PRIVATE
    aja-output.cpp
    aja-output.hpp
    aja-source.cpp
    aja-source.hpp
    audio-repack.c
    audio-repack.h
    audio-repack.hpp
    main.cpp
)

target_compile_options(
  aja-support
  PUBLIC
    $<$<COMPILE_LANG_AND_ID:CXX,AppleClang,Clang,GNU>:-Wno-deprecated-declarations>
    $<$<PLATFORM_ID:Linux,FreeBSD,OpenBSD>:-Wno-unused-variable>
)

target_link_libraries(aja PRIVATE OBS::aja-support)

if(OS_WINDOWS)
  configure_file(cmake/windows/obs-module.rc.in win-aja.rc)
  target_sources(aja PRIVATE win-aja.rc)
  set_property(TARGET aja PROPERTY COMPILE_WARNING_AS_ERROR FALSE)
endif()

set_target_properties_obs(aja PROPERTIES FOLDER plugins/aja PREFIX "")

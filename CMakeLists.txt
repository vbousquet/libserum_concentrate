cmake_minimum_required(VERSION 3.25)

set(PLATFORM "win" CACHE STRING "Platform")
set(ARCH "x64" CACHE STRING "Arch")

option(BUILD_SHARED "Option to build shared library" ON)
option(BUILD_STATIC "Option to build static library" ON)

message(STATUS "PLATFORM: ${PLATFORM}")
message(STATUS "ARCH: ${ARCH}")

message(STATUS "BUILD_SHARED: ${BUILD_SHARED}")
message(STATUS "BUILD_STATIC: ${BUILD_STATIC}")

if(PLATFORM STREQUAL "macos")
   set(CMAKE_OSX_DEPLOYMENT_TARGET 14.0)
elseif(PLATFORM STREQUAL "ios" OR PLATFORM STREQUAL "ios-simulator")
   set(CMAKE_SYSTEM_NAME iOS)
   if (PLATFORM STREQUAL "ios-simulator")
      set(CMAKE_OSX_SYSROOT iphonesimulator)
   endif()
   set(CMAKE_OSX_DEPLOYMENT_TARGET 17.0)
elseif(PLATFORM STREQUAL "tvos")
   set(CMAKE_SYSTEM_NAME tvOS)
   set(CMAKE_OSX_DEPLOYMENT_TARGET 17.0)
elseif(PLATFORM STREQUAL "android")
   set(CMAKE_SYSTEM_NAME Android)
   set(CMAKE_SYSTEM_VERSION 30)
   set(CMAKE_ANDROID_ARCH_ABI arm64-v8a)
endif()


file(READ src/serum-version.h version)
string(REGEX MATCH "SERUM_VERSION_MAJOR[ ]+([0-9]+)" _tmp ${version})
set(VERSION_MAJOR "${CMAKE_MATCH_1}")
string(REGEX MATCH "SERUM_VERSION_MINOR[ ]+([0-9]+)" _tmp ${version})
set(VERSION_MINOR "${CMAKE_MATCH_1}")
string(REGEX MATCH "SERUM_VERSION_PATCH[ ]+([0-9]+)" _tmp ${version})
set(VERSION_PATCH "${CMAKE_MATCH_1}")
string(REGEX MATCH "SERUM_CONCENTRATE_VERSION[ ]+([0-9]+)" _tmp ${version})
set(VERSION_CONCENTRATE "${CMAKE_MATCH_1}")

project(serum VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}"
   DESCRIPTION "Cross-platform library for decoding Serum files, a colorization format for pinball ROMs. Concentrate version: ${VERSION_CONCENTRATE}")

if(PLATFORM STREQUAL "win")
   if(ARCH STREQUAL "x86")
      add_compile_definitions(WIN32)
   endif()
elseif(PLATFORM STREQUAL "macos")
   if (ARCH STREQUAL "arm64")
      set(CMAKE_OSX_ARCHITECTURES arm64)
   elseif(ARCH STREQUAL "x64")
      set(CMAKE_OSX_ARCHITECTURES x86_64)
   endif()
   set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
   set(CMAKE_INSTALL_RPATH "@executable_path")
elseif(PLATFORM STREQUAL "ios" OR PLATFORM STREQUAL "ios-simulator" OR PLATFORM STREQUAL "tvos")
   set(CMAKE_OSX_DEPLOYMENT_TARGET 16.0)
   set(CMAKE_OSX_ARCHITECTURES arm64)
elseif(PLATFORM STREQUAL "linux" OR PLATFORM STREQUAL "android")
   set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
   set(CMAKE_INSTALL_RPATH "$ORIGIN")
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_C_STANDARD 99)

set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_C_VISIBILITY_PRESET hidden)

set(SERUM_SOURCES
   src/serum-decode.h
   src/serum-decode.cpp
   third-party/include/miniz/miniz.h
   third-party/include/miniz/miniz.c
)

set(SERUM_INCLUDE_DIRS
   src
   third-party/include
)

if(BUILD_SHARED)
   add_library(serum_shared SHARED ${SERUM_SOURCES})

   target_include_directories(serum_shared PUBLIC ${SERUM_INCLUDE_DIRS})

   if(PLATFORM STREQUAL "win" AND ARCH STREQUAL "x64")
      set(SERUM_OUTPUT_NAME "serum64")
   else()
      set(SERUM_OUTPUT_NAME "serum")
   endif()

   set_target_properties(serum_shared PROPERTIES
      OUTPUT_NAME ${SERUM_OUTPUT_NAME}
      VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}
   )

   install(TARGETS serum_shared
      LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
   )
   install(FILES
      src/serum.h
      src/serum-decode.h
      DESTINATION ${CMAKE_INSTALL_PREFIX}/include)

   if(PLATFORM STREQUAL "win" OR PLATFORM STREQUAL "macos" OR PLATFORM STREQUAL "linux")
      add_executable(serum_test
         src/test.cpp
      )

      target_link_libraries(serum_test PUBLIC serum_shared)
   endif()
endif()

if(BUILD_STATIC)
   add_library(serum_static STATIC ${SERUM_SOURCES})

   target_include_directories(serum_static PUBLIC ${SERUM_INCLUDE_DIRS})

   if(PLATFORM STREQUAL "win")
      set_target_properties(serum_static PROPERTIES
         OUTPUT_NAME "serum_static"
      )
   else()
      set_target_properties(serum_static PROPERTIES
         OUTPUT_NAME "serum"
      )
   endif()

   install(TARGETS serum_static
      LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
   )
   install(FILES
      src/serum.h
      src/serum-decode.h
      DESTINATION ${CMAKE_INSTALL_PREFIX}/include)

   if(PLATFORM STREQUAL "win" OR PLATFORM STREQUAL "macos" OR PLATFORM STREQUAL "linux")
      add_executable(serum_test_s
         src/test.cpp
      )

      target_link_libraries(serum_test_s PUBLIC serum_static)
   endif()
endif()

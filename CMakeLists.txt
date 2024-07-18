cmake_minimum_required(VERSION 3.17)

project(mruby)

set(MRUBY_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/bundled/mruby CACHE STRING "mruby repo root dir")
set(MRUBY_CONFIG_GENERATE_TARGET_DIR ${CMAKE_CURRENT_BINARY_DIR}/config.rb CACHE STRING "mruby config.rb generate target dir")
set(MRUBY_GEMBOXES "stdlib;stdlib-ext;metaprog" CACHE STRING "enabled mruby gemboxes")
set(MRUBY_CONFIG_NAME "mruby-cmake-configurated" CACHE STRING "mruby build config name")

set(MRUBY_BUILD_TARGET_DIR ${CMAKE_CURRENT_BINARY_DIR}/build)

# generate config.rb
if (CMAKE_C_COMPILER_ID STREQUAL "Clang")
  set(MRUBY_CONF_TOOLCHAIN "clang")
elseif (CMAKE_C_COMPILER_ID STREQUAL "MSVC")
  set(MRUBY_CONF_TOOLCHAIN "visualcpp")
else()
  set(MRUBY_CONF_TOOLCHAIN "gcc")
endif()
set(MRUBY_CONF_GEMBOXES "")
foreach(MRUBY_GEMBOX ${MRUBY_GEMBOXES})
  set(MRUBY_CONF_GEMBOXES "${MRUBY_CONF_GEMBOXES}\n  conf.gembox '${MRUBY_GEMBOX}'")
endforeach()
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/template/config.rb.in ${MRUBY_CONFIG_GENERATE_TARGET_DIR} @ONLY)

# generate builder
if(WIN32)
  set(RAKE_EXECUTABLE "rake.bat")
else()
  set(RAKE_EXECUTABLE "rake")
endif()

add_custom_target(mruby_build 
  COMMAND ${RAKE_EXECUTABLE} MRUBY_CONFIG=${MRUBY_CONFIG_GENERATE_TARGET_DIR} MRUBY_BUILD_DIR=${MRUBY_BUILD_TARGET_DIR}
  WORKING_DIRECTORY ${MRUBY_ROOT}
  )

file(MAKE_DIRECTORY ${MRUBY_BUILD_TARGET_DIR}/${MRUBY_CONFIG_NAME}/include)
add_library(mruby STATIC IMPORTED GLOBAL)
add_dependencies(mruby mruby_build)
set_target_properties(mruby
  PROPERTIES
  IMPORTED_LOCATION ${MRUBY_BUILD_TARGET_DIR}/${MRUBY_CONFIG_NAME}/lib/libmruby.a
  INTERFACE_INCLUDE_DIRECTORIES ${MRUBY_BUILD_TARGET_DIR}/${MRUBY_CONFIG_NAME}/include
  )

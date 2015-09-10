file(GLOB_RECURSE  source_file  "*.c" "*.cpp")

# Should add ../ANT_DLL/*.cpp file into static library
set(source_dll ${ANT_SOURCE_DIR}/src/ANT_DLL/ant.cpp)
list (APPEND source_file ${source_dll})

set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDE_DIR include/ant CACHE PATH
  "Installation directory for header files")
set(INSTALL_CMAKE_DIR lib/cmake/ant CACHE PATH
  "Installation directory for CMake files")

# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
  set(var INSTALL_${p}_DIR)
  if(NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif()
endforeach()

add_library (ant ${source_file})

install (TARGETS ant 
         EXPORT AntTargets 
         RUNTIME DESTINATION bin
         LIBRARY DESTINATION "${INSTALL_LIB_DIR}"
         ARCHIVE DESTINATION "${INSTALL_LIB_DIR}")

install (FILES ${headers}
         DESTINATION include/ant)

message("Current binary dir: ${CMAKE_CURRENT_BINARY_DIR}")

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/AntConfigVersion.cmake"
  VERSION ${ANT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

export(TARGETS ant
       FILE "${CMAKE_CURRENT_BINARY_DIR}/AntTargets.cmake")

file(RELATIVE_PATH REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}"
   "${INSTALL_INCLUDE_DIR}")
message("RELATIVE_PATH: ${REL_INCLUDE_DIR}")

set(CONF_INCLUDE_DIRS "\${ANT_CMAKE_DIR}/${REL_INCLUDE_DIR}")
configure_file(${PROJECT_SOURCE_DIR}/cmake/AntConfig.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/AntConfig.cmake"
  @ONLY
)

install(EXPORT AntTargets
  FILE AntTargets.cmake
  DESTINATION
  ${INSTALL_CMAKE_DIR}
)

install(
  FILES
  "${CMAKE_CURRENT_BINARY_DIR}/AntConfig.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/AntConfigVersion.cmake"
  DESTINATION
  ${INSTALL_CMAKE_DIR}
  COMPONENT
  Devel
)


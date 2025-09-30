cmake_minimum_required(VERSION 3.15)

project ("sqliteFfi LANGUAGES C")

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  MESSAGE(FATAL_ERROR "Toolchain file not set, try using: cmake --preset default")
endif()

add_subdirectory(external/ag-ffi-header)

find_package(unofficial-sqlite3 CONFIG REQUIRED)

add_library (sqliteFfi STATIC sqlite-ffi.c)

set_property(TARGET sqliteFfi PROPERTY C_STANDARD 11)
target_link_libraries(sqliteFfi PRIVATE
    ag_ffi_header
    unofficial::sqlite3::sqlite3)

target_compile_options(sqliteFfi PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/EHs-c->
    $<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-fno-exceptions>
)

set(DST "$<IF:$<BOOL:${AG_TRIPLE}>,${AG_OUT_DIR}/${AG_TRIPLE}/$<CONFIG>,${CMAKE_BINARY_DIR}>")
set(AG_DST "$<IF:$<BOOL:${AG_TRIPLE}>,${AG_OUT_DIR},${CMAKE_BINARY_DIR}>")

set_target_properties(sqliteFfi PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${DST}"
    LIBRARY_OUTPUT_DIRECTORY "${DST}"
    RUNTIME_OUTPUT_DIRECTORY "${DST}"
)

add_custom_command(TARGET sqliteFfi POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_CURRENT_SOURCE_DIR}/sqliteFfi.ag"
            "${AG_DST}/sqliteFfi.ag"
)

if(WIN32)
  if(DEFINED AG_TRIPLE)
    set(DLL_TRIPLE "${AG_TRIPLE}")
  else()
    set(DLL_TRIPLE "${VCPKG_TARGET_TRIPLET}")
  endif()
  if(CMAKE_BUILD_TYPE STREQUAL "Debug" OR CONFIG STREQUAL "Debug")
    find_file(SQLITE3_DLL_DBG NAMES sqlite3.dll PATHS "${CMAKE_BINARY_DIR}/vcpkg_installed/${DLL_TRIPLE}/debug/bin" NO_DEFAULT_PATH)
    add_custom_command(TARGET sqliteFfi POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy_if_different "${SQLITE3_DLL_DBG}" "${DST}/sqlite3.dll"
    )
    find_file(SQLITE3_PDB_DBG NAMES sqlite3.pdb PATHS "${CMAKE_BINARY_DIR}/vcpkg_installed/${DLL_TRIPLE}/debug/bin" NO_DEFAULT_PATH)
    add_custom_command(TARGET sqliteFfi POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy_if_different "${SQLITE3_PDB_DBG}" "${DST}/sqlite3.pdb"
    )
  else()
    find_file(SQLITE3_DLL_REL NAMES sqlite3.dll PATHS "${CMAKE_BINARY_DIR}/vcpkg_installed/${DLL_TRIPLE}/bin" NO_DEFAULT_PATH)
    add_custom_command(TARGET sqliteFfi POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy_if_different "${SQLITE3_DLL_REL}" "${DST}/sqlite3.dll"
    )
  endif()
endif()

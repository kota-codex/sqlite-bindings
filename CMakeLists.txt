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

if(AG_TRIPLE) # called from superbuild
  message(STATUS "sqliteFfi: using superbuild triple = ${AG_TRIPLE}")

  set_target_properties(sqliteFfi PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/out/${AG_TRIPLE}/$<CONFIG>"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/out/${AG_TRIPLE}/$<CONFIG>"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/out/${AG_TRIPLE}/$<CONFIG>"
  )

  add_custom_command(TARGET sqliteFfi POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_CURRENT_SOURCE_DIR}/sqliteFfi.ag"
            "${CMAKE_BINARY_DIR}/out/sqliteFfi.ag")
else()
  message(STATUS "sqliteFfi: standalone build, standard layout")

  add_custom_command(TARGET sqliteFfi POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_CURRENT_SOURCE_DIR}/sqliteFfi.ag"
            "${CMAKE_BINARY_DIR}/sqliteFfi.ag")
endif()

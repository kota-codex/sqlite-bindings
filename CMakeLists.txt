cmake_minimum_required(VERSION 3.15)

project (sqliteFfi LANGUAGES C)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  MESSAGE(FATAL_ERROR "Toolchain file not set, try using: cmake --preset default")
endif()

add_subdirectory(ag-ffi-header)

find_package(unofficial-sqlite3 CONFIG REQUIRED)

add_library (sqliteFfi STATIC sqlite-ffi.c)

target_link_libraries(sqliteFfi PUBLIC unofficial::sqlite3::sqlite3)

ag_ffi_configure_target(sqliteFfi sqliteFfi.ag)

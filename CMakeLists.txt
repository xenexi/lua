cmake_minimum_required(VERSION 3.8)
project(lua LANGUAGES C VERSION 5.4.3)

option(LUA_SUPPORT_DL "Support dynamic loading of compiled modules" OFF)

FILE(GLOB LUA_SRC *.c)
list(FILTER LUA_SRC EXCLUDE REGEX ".*((ltests.c)|(lua.c)|(onelua.c))")
FILE(GLOB LUA_HEADER *.h)

add_library(lua_static STATIC ${LUA_SRC})
set_target_properties(lua_static PROPERTIES OUTPUT_NAME "lua")
target_include_directories(lua_static PUBLIC lua)

if (UNIX)
    set(LUA_DEFINITIONS)
    if (NOT APPLE AND NOT EMSCRIPTEN)
        find_library(M_LIBRARY m)
    else()
        set(M_LIBRARY "")
    endif()

    list(APPEND LUA_DEFINITIONS LUA_USE_POSIX)
    if (LUA_SUPPORT_DL)
        target_compile_definitions(lua_static PRIVATE "LUA_USE_DLOPEN")
        target_link_libraries(lua_static INTERFACE dl)
    endif()

    target_compile_definitions(lua_static PUBLIC "${LUA_DEFINITIONS}")
    target_compile_options(lua_static PRIVATE "-Wall" "-Wextra")
endif()

add_library(lua::lua ALIAS lua_static)
target_sources(lua_static INTERFACE ${LUA_HEADER})
set_target_properties(lua_static
	PROPERTIES
	EXPORT_NAME lua::lua)

# # Version configurations
#configure_package_config_file(
#	cmake/sol2-config.cmake.in
#	"${CMAKE_CURRENT_BINARY_DIR}/cmake/sol2-config.cmake"
#	INSTALL_DESTINATION lib/cmake/sol2
#	NO_CHECK_REQUIRED_COMPONENTS_MACRO)
#
#write_basic_package_version_file(
#	"${CMAKE_CURRENT_BINARY_DIR}/cmake/sol2-config-version.cmake"
#	COMPATIBILITY AnyNewerVersion)
#
export(TARGETS lua_static FILE "${CMAKE_CURRENT_BINARY_DIR}/cmake/lua-targets.cmake")



cmake_minimum_required(VERSION 3.5)

project(libuuid-cmake LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if (NOT UNIX)
    message(FATAL_ERROR "unsupporsted ${CMAKE_SYSTEM_NAME}")
endif()

set (install_prefix "${CMAKE_CURRENT_BINARY_DIR}/local")

set(THIRDPARTY_CURL_PREFIX ${install_prefix} PARENT_SCOPE)
set(HAS_THIRDPARTY_CURL ON PARENT_SCOPE)

include(ExternalProject)

function(libUuidMain)

    add_library(thirdparty_uuid INTERFACE)
    add_library(thirdparty_uuid_static STATIC IMPORTED GLOBAL)
    if (NOT CMD_MAKE)
        message(STATUS "[curl] set CMD_MAKE is make.")
        set(CMD_MAKE make)
    endif()

    message(STATUS "[curl] add external project: ${PROJECT_NAME}")

    # 依赖库 https://curl.se/docs/libs.html
    set(configure_command
        perl ./configure
        "--prefix=${install_prefix}"
        "CFLAGS=-fPIC"
        "CPPFLAGS=-fPIC"
        )

    ExternalProject_Add(
        libuuid
        URL "http://sourceforge.net/projects/libuuid/files/libuuid-1.0.3.tar.gz"
        URL_HASH "SHA256=46af3275291091009ad7f1b899de3d0cea0252737550e7919d17237997db5644"
        CONFIGURE_COMMAND ${configure_command}
        BUILD_COMMAND ${CMD_MAKE}
        BUILD_IN_SOURCE 1
        INSTALL_COMMAND ${CMD_MAKE} install
        UPDATE_COMMAND ""
        )
    add_dependencies(thirdparty_uuid_static libuuid)

    set_target_properties(thirdparty_uuid_static PROPERTIES IMPORTED_LOCATION
        "${install_prefix}/lib/libuuid.a"
        )

    target_link_libraries(thirdparty_uuid INTERFACE thirdparty_uuid_static)
    if (CMAKE_SYSTEM_NAME MATCHES "Darwin")
        target_link_libraries(thirdparty_uuid INTERFACE "-framework SystemConfiguration" dl z)
    endif()
    target_include_directories(thirdparty_uuid INTERFACE "${install_prefix}/include")

endfunction()

libUuidMain()

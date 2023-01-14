cmake_minimum_required(VERSION 3.25)

project(
    "SomeLibrary"
    # ...
)

# if (NOT PROJECT_IS_TOP_LEVEL)
#     message(STATUS "This project has a top-level one called [${CMAKE_PROJECT_NAME}]")
# else()
#     message(STATUS "This project is a top-level one")
# endif()

if (NOT CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    message(STATUS "This project has a top-level one called [${CMAKE_PROJECT_NAME}]")
else()
    message(STATUS "This project is a top-level one")
endif()

# ----

add_library(${PROJECT_NAME} STATIC)

target_sources(${PROJECT_NAME}
    PRIVATE
        src/some.cpp
)

# ----

target_include_directories(${PROJECT_NAME}
    PRIVATE
        # Ref: the library itself -> the internal headers
        ${CMAKE_CURRENT_SOURCE_DIR}/src
    PUBLIC
        # Ref: the library itself -> the public headers
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        # Ref: external projects -> the public headers
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>

)

# ----

if(DEFINED CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    message(
        STATUS
        "CMAKE_INSTALL_PREFIX is not set\n"
        "Default value: ${CMAKE_INSTALL_PREFIX}\n"
        "Will set it to ${CMAKE_SOURCE_DIR}/install"
    )
    set(CMAKE_INSTALL_PREFIX
        "${CMAKE_SOURCE_DIR}/install"
        CACHE PATH "Where the library will be install to" FORCE
    )
else()
    message(
        STATUS
        "CMAKE_INSTALL_PREFIX was already set\n"
        "Current value: ${CMAKE_INSTALL_PREFIX}"
    )
endif()

# ----

set(CMAKE_CUSTOM_PUBLIC_HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/include/some.h
)

set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${CMAKE_CUSTOM_PUBLIC_HEADERS}")

set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX "d")

# ----

include(GNUInstallDirs)

install(TARGETS ${PROJECT_NAME}
    EXPORT "${PROJECT_NAME}Targets"
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

set(CMAKE_CUSTOM_NAMESPACE some)

install(EXPORT "${PROJECT_NAME}Targets"
    FILE "${PROJECT_NAME}Targets.cmake"
    NAMESPACE ${CMAKE_CUSTOM_NAMESPACE}::
    DESTINATION cmake
)

include(CMakePackageConfigHelpers)

set(CMAKE_CUSTOM_PACKAGE_VERSION "0.1.0")

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION "${CMAKE_CUSTOM_PACKAGE_VERSION}"
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION  cmake
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION cmake
)

export(EXPORT "${PROJECT_NAME}Targets"
    FILE "${CMAKE_CURRENT_BINARY_DIR}/cmake/${PROJECT_NAME}Targets.cmake"
    NAMESPACE ${namespace}::
)
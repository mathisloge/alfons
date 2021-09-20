cmake_minimum_required(VERSION 3.12)
project(alfons VERSION 1.0.0)
if (APPLE)
  option(HB_HAVE_CORETEXT "Enable CoreText shaper backend on macOS" ON)
endif ()
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")

set(CMAKE_DEBUG_POSTFIX "d")

option(BUILD_SHARED_LIBS "build shared" ON)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/src)

# Install alfons
install(DIRECTORY "${PROJECT_SOURCE_DIR}/src/" TYPE INCLUDE FILES_MATCHING PATTERN  "*.h")
install(TARGETS linebreak alfons
    EXPORT alfonsTargets
)
install(EXPORT alfonsTargets
    FILE alfonsTargets.cmake
    DESTINATION lib/cmake/alfons
)
include(CMakePackageConfigHelpers)
configure_package_config_file(${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/alfonsConfig.cmake
    INSTALL_DESTINATION lib/cmake/alfons
)
write_basic_package_version_file(
    alfonsConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
)
install(EXPORT alfonsTargets
    FILE alfonsTargets.cmake
    NAMESPACE alfons::
    DESTINATION lib/cmake/alfons
)
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/alfonsConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/alfonsConfigVersion.cmake"
    DESTINATION lib/cmake/alfons
)
